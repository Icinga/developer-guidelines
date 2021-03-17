
## Cluster <a id="technical-concepts-cluster"></a>

This documentation refers to technical roles between cluster
endpoints.

- The `server` or `parent` role accepts incoming connection attempts and handles requests
- The `client` role actively connects to remote endpoints receiving config/commands, requesting certificates, etc.

A client role is not necessarily bound to the Icinga agent.
It may also be a satellite which actively connects to the
master.

### Communication <a id="technical-concepts-cluster-communication"></a>

Icinga 2 uses its own certificate authority (CA) by default. The
public and private CA keys can be generated on the signing master.

Each node certificate must be signed by the private CA key.

Note: The following description uses `parent node` and `child node`.
This also applies to nodes in the same cluster zone.

During the connection attempt, a TLS handshake is performed.
If the public certificate of a child node is not signed by the same
CA, the child node is not trusted and the connection will be closed.

If the TLS handshake succeeds, the parent node reads the
certificate's common name (CN) of the child node and looks for
a local Endpoint object name configuration.

If there is no Endpoint object found, further communication
(runtime and config sync, etc.) is terminated.

The child node also checks the CN from the parent node's public
certificate. If the child node does not find any local Endpoint
object name configuration, it will not trust the parent node.

Both checks prevent accepting cluster messages from an untrusted
source endpoint.

If an Endpoint match was found, there is one additional security
mechanism in place: Endpoints belong to a Zone hierarchy.

Several cluster messages can only be sent "top down", others like
check results are allowed being sent from the child to the parent node.

Once this check succeeds the cluster messages are exchanged and processed.


### CSR Signing <a id="technical-concepts-cluster-csr-signing"></a>

In order to make things easier, Icinga 2 provides built-in methods
to allow child nodes to request a signed certificate from the
signing master.

Icinga 2 v2.8 introduces the possibility to request certificates
from indirectly connected nodes. This is required for multi level
cluster environments with masters, satellites and agents.

CSR Signing in general starts with the master setup. This step
ensures that the master is in a working CSR signing state with:

* public and private CA key in `/var/lib/icinga2/ca`
* private `TicketSalt` constant defined inside the `api` feature
* Cluster communication is ready and Icinga 2 listens on port 5665

The child node setup which is run with CLI commands will now
attempt to connect to the parent node. This is not necessarily
the signing master instance, but could also be a parent satellite node.

During this process the child node asks the user to verify the
parent node's public certificate to prevent MITM attacks.

There are two methods to request signed certificates:

* Add the ticket into the request. This ticket was generated on the master
  beforehand and contains hashed details for which client it has been created.
  The signing master uses this information to automatically sign the certificate
  request.

* Do not add a ticket into the request. It will be sent to the signing master
  which stores the pending request. Manual user interaction with CLI commands
  is necessary to sign the request.

The certificate request is sent as `pki::RequestCertificate` cluster
message to the parent node.

If the parent node is not the signing master, it stores the request
in `/var/lib/icinga2/certificate-requests` and forwards the
cluster message to its parent node.

Once the message arrives on the signing master, it first verifies that
the sent certificate request is valid. This is to prevent unwanted errors
or modified requests from the "proxy" node.

After verification, the signing master checks if the request contains
a valid signing ticket. It hashes the certificate's common name and
compares the value to the received ticket number.

If the ticket is valid, the certificate request is immediately signed
with CA key. The request is sent back to the client inside a `pki::UpdateCertificate`
cluster message.

If the child node was not the certificate request origin, it only updates
the cached request for the child node and send another cluster message
down to its child node (e.g. from a satellite to an agent).


If no ticket was specified, the signing master waits until the
`ca sign` CLI command manually signed the certificate.

> **Note**
>
> Push notifications for manual request signing is not yet implemented (TODO).

Once the child node reconnects it synchronizes all signed certificate requests.
This takes some minutes and requires all nodes to reconnect to each other.


#### CSR Signing: Clients without parent connection <a id="technical-concepts-cluster-csr-signing-clients-no-connection"></a>

There is an additional scenario: The setup on a child node does
not necessarily need a connection to the parent node.

This mode leaves the node in a semi-configured state. You need
to manually copy the master's public CA key into `/var/lib/icinga2/certs/ca.crt`
on the client before starting Icinga 2.

> **Note**
>
> The `client` in this case can be either a satellite or an agent.

The parent node needs to actively connect to the child node.
Once this connections succeeds, the child node will actively
request a signed certificate.

The update procedure works the same way as above.

### High Availability <a id="technical-concepts-cluster-ha"></a>

General high availability is automatically enabled between two endpoints in the same
cluster zone.

**This requires the same configuration and enabled features on both nodes.**

HA zone members trust each other and share event updates as cluster messages.
This includes for example check results, next check timestamp updates, acknowledgements
or notifications.

This ensures that both nodes are synchronized. If one node goes away, the
remaining node takes over and continues as normal.

#### High Availability: Object Authority <a id="technical-concepts-cluster-ha-object-authority"></a>

Cluster nodes automatically determine the authority for configuration
objects. By default, all config objects are set to `HARunEverywhere` and
as such the object authority is true for any config object on any instance.

Specific objects can override and influence this setting, e.g. with `HARunOnce`
instead prior to config object activation.

This is done when the daemon starts and in a regular interval inside
the ApiListener class, specifically calling `ApiListener::UpdateObjectAuthority()`.

The algorithm works like this:

* Determine whether this instance is assigned to a local zone and endpoint.
* Collects all endpoints in this zone if they are connected.
* If there's two endpoints, but only us seeing ourselves and the application start is less than 60 seconds in the past, do nothing (wait for cluster reconnect to take place, grace period).
* Sort the collected endpoints by name.
* Iterate over all config types and their respective objects
* Ignore !active objects
* Ignore objects which are !HARunOnce. This means, they can run multiple times in a zone and don't need an authority update.
* If this instance doesn't have a local zone, set authority to true. This is for non-clustered standalone environments where everything belongs to this instance.
* Calculate the object authority based on the connected endpoint names.
* Set the authority (true or false)

The object authority calculation works "offline" without any message exchange.
Each instance alculates the SDBM hash of the config object name, puts that in contrast
modulo the connected endpoints size.
This index is used to lookup the corresponding endpoint in the connected endpoints array,
including the local endpoint. Whether the local endpoint is equal to the selected endpoint,
or not, this sets the authority to `true` or `false`.

```cpp
authority = endpoints[Utility::SDBM(object->GetName()) % endpoints.size()] == my_endpoint;
```

`ConfigObject::SetAuthority(bool authority)` triggers the following events:

* Authority is true and object now paused: Resume the object and set `paused` to `false`.
* Authority is false, object not paused: Pause the object and set `paused` to true.

**This results in activated but paused objects on one endpoint.** You can verify
that by querying the `paused` attribute for all objects via REST API
or debug console on both endpoints.

Endpoints inside a HA zone calculate the object authority independent from each other.
This object authority is important for selected features explained below.

Since features are configuration objects too, you must ensure that all nodes
inside the HA zone share the same enabled features. If configured otherwise,
one might have a checker feature on the left node, nothing on the right node.
This leads to late check results because one half is not executed by the right
node which holds half of the object authorities.

By default, features are enabled to "Run-Everywhere". Specific features which
support HA awareness, provide the `enable_ha` configuration attribute. When `enable_ha`
is set to `true` (usually the default), "Run-Once" is set and the feature pauses on one side.

```
vim /etc/icinga2/features-enabled/graphite.conf

object GraphiteWriter "graphite" {
  ...
  enable_ha = true
}
```

Once such a feature is paused, there won't be any more event handling, e.g. the Elasticsearch
feature won't process any checkresults nor write to the Elasticsearch REST API.

When the cluster connection drops, the feature configuration object is updated with
the new object authority by the ApiListener timer and resumes its operation. You can see
that by grepping the log file for `resumed` and `paused`.

```
[2018-10-24 13:28:28 +0200] information/GraphiteWriter: 'g-ha' paused.
```

```
[2018-10-24 13:28:28 +0200] information/GraphiteWriter: 'g-ha' resumed.
```

Specific features with HA capabilities are explained below.

#### High Availability: Checker <a id="technical-concepts-cluster-ha-checker"></a>

The `checker` feature only executes checks for `Checkable` objects (Host, Service)
where it is authoritative.

That way each node only executes checks for a segment of the overall configuration objects.

The cluster message routing ensures that all check results are synchronized
to nodes which are not authoritative for this configuration object.


#### High Availability: Notifications <a id="technical-concepts-cluster-notifications"></a>

The `notification` feature only sends notifications for `Notification` objects
where it is authoritative.

That way each node only executes notifications for a segment of all notification objects.

Notified users and other event details are synchronized throughout the cluster.
This is required if for example the DB IDO feature is active on the other node.

#### High Availability: DB IDO <a id="technical-concepts-cluster-ha-ido"></a>

If you don't have HA enabled for the IDO feature, both nodes will
write their status and historical data to their own separate database
backends.

In order to avoid data separation and a split view (each node would require its
own Icinga Web 2 installation on top), the high availability option was added
to the DB IDO feature. This is enabled by default with the `enable_ha` setting.

This requires a central database backend. Best practice is to use a MySQL cluster
with a virtual IP.

Both Icinga 2 nodes require the connection and credential details configured in
their DB IDO feature.

During startup Icinga 2 calculates whether the feature configuration object
is authoritative on this node or not. The order is an alpha-numeric
comparison, e.g. if you have `master1` and `master2`, Icinga 2 will enable
the DB IDO feature on `master2` by default.

If the connection between endpoints drops, the object authority is re-calculated.

In order to prevent data duplication in a split-brain scenario where both
nodes would write into the same database, there is another safety mechanism
in place.

The split-brain decision which node will write to the database is calculated
from a quorum inside the `programstatus` table. Each node
verifies whether the `endpoint_name` column is not itself on database connect.
In addition to that the DB IDO feature compares the `last_update_time` column
against the current timestamp plus the configured `failover_timeout` offset.

That way only one active DB IDO feature writes to the database, even if they
are not currently connected in a cluster zone. This prevents data duplication
in historical tables.

### Health Checks <a id="technical-concepts-cluster-health-checks"></a>

#### cluster-zone <a id="technical-concepts-cluster-health-checks-cluster-zone"></a>

This built-in check provides the possibility to check for connectivity between
zones.

If you for example need to know whether the `master` zone is connected and processing
messages with the child zone called `satellite` in this example, you can configure
the [cluster-zone](10-icinga-template-library.md#itl-icinga-cluster-zone) check as new service on all `master` zone hosts.

```
vim /etc/zones.d/master/host1.conf

object Service "cluster-zone-satellite" {
  check_command = "cluster-zone"
  host_name = "host1"

  vars.cluster_zone = "satellite"
}
```

The check itself changes to NOT-OK if one or more child endpoints in the child zone
are not connected to parent zone endpoints.

In addition to the overall connectivity check, the log lag is calculated based
on the to-be-sent replay log. Each instance stores that for its configured endpoint
objects.

This health check iterates over the target zone (`cluster_zone`) and their endpoints.

The log lag is greater than zero if

* the replay log synchronization is in progress and not yet finished or
* the endpoint is not connected, and no replay log sync happened (obviously).

The final log lag value is the worst value detected. If satellite1 has a log lag of
`1.5` and satellite2 only has `0.5`, the computed value will be `1.5.`.

You can control the check state by using optional warning and critical thresholds
for the log lag value.

If this service exists multiple times, e.g. for each master host object, the log lag
may differ based on the execution time. This happens for example on restart of
an instance when the log replay is in progress and a health check is executed at different
times.
If the endpoint is not connected, both master instances may have saved a different log replay
position from the last synchronisation.

The lag value is returned as performance metric key `slave_lag`.

Icinga 2 v2.9+ adds more performance metrics for these values:

* `last_messages_sent` and `last_messages_received` as UNIX timestamp
* `sum_messages_sent_per_second` and `sum_messages_received_per_second`
* `sum_bytes_sent_per_second` and `sum_bytes_received_per_second`


### Config Sync <a id="technical-concepts-cluster-config-sync"></a>

The visible feature for the user is to put configuration files in `/etc/icinga2/zones.d/<zonename>`
and have them synced automatically to all involved zones and endpoints.

This not only includes host and service objects being checked
in a satellite zone, but also additional config objects such as
commands, groups, timeperiods and also templates.

Additional thoughts and complexity added:

- Putting files into zone directory names removes the burden to set the `zone` attribute on each object in this directory. This is done automatically by the config compiler.
- Inclusion of `zones.d` happens automatically, the user shouldn't be bothered about this.
- Before the REST API was created, only static configuration files in `/etc/icinga2/zones.d` existed. With the addition of config packages, additional `zones.d` targets must be registered (e.g. used by the Director)
- Only one config master is allowed. This one identifies itself with configuration files in `/etc/icinga2/zones.d`. This is not necessarily the zone master seen in the debug logs, that one is important for message routing internally.
- Objects and templates which cannot be bound into a specific zone (e.g. hosts in the satellite zone) must be made available "globally".
- Users must be able to deny the synchronisation of specific zones, e.g. for security reasons.

#### Config Sync: Config Master <a id="technical-concepts-cluster-config-sync-config-master"></a>

All zones must be configured and included in the `zones.conf` config file beforehand.
The zone names are the identifier for the directories underneath the `/etc/icinga2/zones.d`
directory. If a zone is not configured, it will not be included in the config sync - keep this
in mind for troubleshooting.

When the config master starts, the content of `/etc/icinga2/zones.d` is automatically
included. There's no need for an additional entry in `icinga2.conf` like `conf.d`.
You can verify this by running the config validation on debug level:

```
icinga2 daemon -C -x debug | grep 'zones.d'

[2019-06-19 15:16:19 +0200] notice/ConfigCompiler: Compiling config file: /etc/icinga2/zones.d/global-templates/commands.conf
```

Once the config validation succeeds, the startup routine for the daemon
copies the files into the "production" directory in `/var/lib/icinga2/api/zones`.
This directory is used for all endpoints where Icinga stores the received configuration.
With the exception of the config master retrieving this from `/etc/icinga2/zones.d` instead.

These operations are logged for better visibility.

```
[2019-06-19 15:26:38 +0200] information/ApiListener: Copying 1 zone configuration files for zone 'global-templates' to '/var/lib/icinga2/api/zones/global-templates'.
[2019-06-19 15:26:38 +0200] information/ApiListener: Updating configuration file: /var/lib/icinga2/api/zones/global-templates//_etc/commands.conf
```

The master is finished at this point. Depending on the cluster configuration,
the next iteration is a connected endpoint after successful TLS handshake and certificate
authentication.

It calls `SendConfigUpdate(client)` which sends the [config::Update](19-technical-concepts.md#technical-concepts-json-rpc-messages-config-update)
JSON-RPC message including all required zones and their configuration file content.


#### Config Sync: Receive Config <a id="technical-concepts-cluster-config-sync-receive-config"></a>

The secondary master endpoint and endpoints in a child zone will be connected to the config
master. The endpoint receives the [config::Update](19-technical-concepts.md#technical-concepts-json-rpc-messages-config-update)
JSON-RPC message and processes the content in `ConfigUpdateHandler()`. This method checks
whether config should be accepted. In addition to that, it locks a local mutex to avoid race conditions
with multiple syncs in parallel.

After that, the received configuration content is analysed.

> **Note**
>
> The cluster design allows that satellite endpoints may connect to the secondary master first.
> There is no immediate need to always connect to the config master first, especially since
> the satellite endpoints don't know that.
>
> The secondary master not only stores the master zone config files, but also all child zones.
> This is also the case for any HA enabled zone with more than one endpoint.


2.11 puts the received configuration files into a staging directory in
`/var/lib/icinga2/api/zones-stage`. Previous versions directly wrote the
files into production which could have led to broken configuration on the
next manual restart.

```
[2019-06-19 16:08:29 +0200] information/ApiListener: New client connection for identity 'master1' to [127.0.0.1]:5665
[2019-06-19 16:08:30 +0200] information/ApiListener: Applying config update from endpoint 'master1' of zone 'master'.
[2019-06-19 16:08:30 +0200] information/ApiListener: Received configuration for zone 'agent' from endpoint 'master1'. Comparing the checksums.
[2019-06-19 16:08:30 +0200] information/ApiListener: Stage: Updating received configuration file '/var/lib/icinga2/api/zones-stage/agent//_etc/host.conf' for zone 'agent'.
[2019-06-19 16:08:30 +0200] information/ApiListener: Applying configuration file update for path '/var/lib/icinga2/api/zones-stage/agent' (176 Bytes).
[2019-06-19 16:08:30 +0200] information/ApiListener: Received configuration for zone 'master' from endpoint 'master1'. Comparing the checksums.
[2019-06-19 16:08:30 +0200] information/ApiListener: Applying configuration file update for path '/var/lib/icinga2/api/zones-stage/master' (17 Bytes).
[2019-06-19 16:08:30 +0200] information/ApiListener: Received configuration from endpoint 'master1' is different to production, triggering validation and reload.
```

It then validates the received configuration in its own config stage. There is
an parameter override in place which disables the automatic inclusion of the production
config in `/var/lib/icinga2/api/zones`.

Once completed, the reload is triggered. This follows the same configurable timeout
as with the global reload.

```
[2019-06-19 16:52:26 +0200] information/ApiListener: Config validation for stage '/var/lib/icinga2/api/zones-stage/' was OK, replacing into '/var/lib/icinga2/api/zones/' and triggering reload.
[2019-06-19 16:52:27 +0200] information/Application: Got reload command: Started new instance with PID '19945' (timeout is 300s).
[2019-06-19 16:52:28 +0200] information/Application: Reload requested, letting new process take over.
```

Whenever the staged configuration validation fails, Icinga logs this including a reference
to the startup log file which includes additional errors.

```
[2019-06-19 15:45:27 +0200] critical/ApiListener: Config validation failed for staged cluster config sync in '/var/lib/icinga2/api/zones-stage/'. Aborting. Logs: '/var/lib/icinga2/api/zones-stage//startup.log'
```


#### Config Sync: Changes and Reload <a id="technical-concepts-cluster-config-sync-changes-reload"></a>

Whenever a new configuration is received, it is validated and upon success, the
daemon automatically reloads. While the daemon continues with checks, the reload
cannot hand over open TCP connections. That being said, reloading the daemon everytime
a configuration is synchronized would lead into many not connected endpoints.

Therefore the cluster config sync checks whether the configuration files actually
changed, and will only trigger a reload when such a change happened.

2.11 calculates a checksum from each file content and compares this to the
production configuration. Previous versions used additional metadata with timestamps from
files which sometimes led to problems with asynchronous dates.

> **Note**
>
> For compatibility reasons, the timestamp metadata algorithm is still intact, e.g.
> when the client is 2.11 already, but the parent endpoint is still on 2.10.

Icinga logs a warning when this happens.

```
Received configuration update without checksums from parent endpoint satellite1. This behaviour is deprecated. Please upgrade the parent endpoint to 2.11+
```


The debug log provides more details on the actual checksums and checks. Future output
may change, use this solely for troubleshooting and debugging whenever the cluster
config sync fails.

```
[2019-06-19 16:13:16 +0200] information/ApiListener: Received configuration for zone 'agent' from endpoint 'master1'. Comparing the checksums.
[2019-06-19 16:13:16 +0200] debug/ApiListener: Checking for config change between stage and production. Old (3): '{"/.checksums":"7ede1276a9a32019c1412a52779804a976e163943e268ec4066e6b6ec4d15d73","/.timestamp":"ec4354b0eca455f7c2ca386fddf5b9ea810d826d402b3b6ac56ba63b55c2892c","/_etc/host.conf":"35d4823684d83a5ab0ca853c9a3aa8e592adfca66210762cdf2e54339ccf0a44"}' vs. new (3): '{"/.checksums":"84a586435d732327e2152e7c9b6d85a340cc917b89ae30972042f3dc344ea7cf","/.timestamp":"0fd6facf35e49ab1b2a161872fa7ad794564eba08624373d99d31c32a7a4c7d3","/_etc/host.conf":"0d62075e89be14088de1979644b40f33a8f185fcb4bb6ff1f7da2f63c7723fcb"}'.
[2019-06-19 16:13:16 +0200] debug/ApiListener: Checking /_etc/host.conf for checksum: 35d4823684d83a5ab0ca853c9a3aa8e592adfca66210762cdf2e54339ccf0a44
[2019-06-19 16:13:16 +0200] debug/ApiListener: Path '/_etc/host.conf' doesn't match old checksum '0d62075e89be14088de1979644b40f33a8f185fcb4bb6ff1f7da2f63c7723fcb' with new checksum '35d4823684d83a5ab0ca853c9a3aa8e592adfca66210762cdf2e54339ccf0a44'.
```


#### Config Sync: Trust <a id="technical-concepts-cluster-config-sync-trust"></a>

The config sync follows the "top down" approach, where the master endpoint in the master
zone is allowed to synchronize configuration to the child zone, e.g. the satellite zone.

Endpoints in the same zone, e.g. a secondary master, receive configuration for the same
zone and all child zones.

Endpoints in the satellite zone trust the parent zone, and will accept the pushed
configuration via JSON-RPC cluster messages. By default, this is disabled and must
be enabled with the `accept_config` attribute in the ApiListener feature (manually or with CLI
helpers).

The satellite zone will not only accept zone configuration for its own zone, but also
all configured child zones. That is why it is important to configure the zone hierarchy
on the satellite as well.

Child zones are not allowed to sync configuration up to the parent zone. Each Icinga instance
evaluates this in startup and knows on endpoint connect which config zones need to be synced.


Global zones have a special trust relationship: They are synced to all child zones, be it
a satellite zone or agent zone. Since checkable objects such as a Host or a Service object
must have only one endpoint as authority, they cannot be put into a global zone (denied by
the config compiler).

Apply rules and templates are allowed, since they are evaluated in the endpoint which received
the synced configuration. Keep in mind that there may be differences on the master and the satellite
when e.g. hostgroup membership is used for assign where expressions, but the groups are only
available on the master.


### Cluster: Message Routing <a id="technical-concepts-cluster-message-routing"></a>

One fundamental part of the cluster message routing is the MessageOrigin object.
This is created when a new JSON-RPC message is received in `JsonRpcConnection::MessageHandler()`.

It contains

- FromZone being extracted from the endpoint object which owns the JsonRpcConnection
- FromClient being the JsonRpcConnection bound to the endpoint object

These attributes are checked in message receive api handlers for security access. E.g. whether a
message origin is from a child zone which is not allowed, etc.
This is explained in the [JSON-RPC messages](19-technical-concepts.md#technical-concepts-json-rpc-messages) chapter.

Whenever such a message is processed on the client, it may trigger additional cluster events
which are sent back to other endpoints. Therefore it is key to always pass the MessageOrigin
`origin` when processing these messages locally.

Example:

- Client receives a CheckResult from another endpoint in the same zone, call it `sender` for now
- Calls ProcessCheckResult() to store the CR and calculcate states, notifications, etc.
- Calls the OnNewCheckResult() signal to trigger IDO updates

OnNewCheckResult() also calls a registered cluster handler which forwards the CheckResult to other cluster members.

Without any origin details, this CheckResult would be relayed to the `sender` endpoint again.
Which processes the message, ProcessCheckResult(), OnNewCheckResult(), sends back and so on.

That creates a loop which our cluster protocol needs to prevent at all cost.

RelayMessageOne() takes care of the routing. This involves fetching the targetZone for this message and its endpoints.

- Don't relay messages to ourselves.
- Don't relay messages to disconnected endpoints.
- Don't relay the message to the zone through more than one endpoint unless this is our own zone.
- Don't relay messages back to the endpoint which we got the message from. **THIS**
- Don't relay messages back to the zone which we got the message from.
- Only relay message to the zone master if we're not currently the zone master.

```
 e1 is zone master, e2 and e3 are zone members.

 Message is sent from e2 or e3:
   !isMaster == true
   targetEndpoint e1 is zone master -> send the message
   targetEndpoint e3 is not zone master -> skip it, avoid routing loops

 Message is sent from e1:
   !isMaster == false -> send the messages to e2 and e3 being the zone routing master.
```

With passing the `origin` the following condition prevents sending a message back to sender:

```cpp
if (origin && origin->FromClient && targetEndpoint == origin->FromClient->GetEndpoint()) {
```

This message then simply gets skipped for this specific Endpoint and is never sent.

This analysis originates from a long-lasting [downtime loop bug](https://github.com/Icinga/icinga2/issues/7198).
