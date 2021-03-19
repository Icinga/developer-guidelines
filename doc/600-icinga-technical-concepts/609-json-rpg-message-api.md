**The JSON-RPC message API is not a public API for end users.** In case you want
to interact with Icinga, use the [REST API](12-icinga2-api.md#icinga2-api).

This section describes the internal cluster messages exchanged between endpoints.

> **Tip**
>
> Debug builds with `icinga2 daemon -DInternal.DebugJsonRpc=1` unveils the JSON-RPC messages.

### Registered Handler Functions

Functions by example:

Event Sender: `Checkable::OnNewCheckResult`

```
On<xyz>.connect(&xyzHandler)
```

Event Receiver (Client): `CheckResultAPIHandler` in `REGISTER_APIFUNCTION`

```
<xyz>APIHandler()
```

### Messages

#### icinga::Hello <a id="technical-concepts-json-rpc-messages-icinga-hello"></a>

> Location: `apilistener.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | icinga::Hello
params    | Dictionary

##### Params

Key                  | Type        | Description
---------------------|-------------|------------------
capabilities         | Number      | Bitmask, see `lib/remote/apilistener.hpp`.
version              | Number      | Icinga 2 version, e.g. 21300 for v2.13.0.

##### Functions

Event Sender: When a new client connects in `NewClientHandlerInternal()`.
Event Receiver: `HelloAPIHandler`

##### Permissions

None, this is a required message.

#### event::Heartbeat <a id="technical-concepts-json-rpc-messages-event-heartbeat"></a>

> Location: `jsonrpcconnection-heartbeat.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::Heartbeat
params    | Dictionary

##### Params

Key       | Type          | Description
----------|---------------|------------------
timeout   | Number        | Heartbeat timeout, sender sets 120s.


##### Functions

Event Sender: `JsonRpcConnection::HeartbeatTimerHandler`
Event Receiver: `HeartbeatAPIHandler`

Both sender and receiver exchange this heartbeat message. If the sender detects
that a client endpoint hasn't sent anything in the updated timeout span, it disconnects
the client. This is to avoid stale connections with no message processing.

##### Permissions

None, this is a required message.

#### event::CheckResult <a id="technical-concepts-json-rpc-messages-event-checkresult"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::CheckResult
params    | Dictionary

##### Params

Key       | Type          | Description
----------|---------------|------------------
host      | String        | Host name
service   | String        | Service name
cr        | Serialized CR | Check result

##### Functions

Event Sender: `Checkable::OnNewCheckResult`
Event Receiver: `CheckResultAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Hosts/services do not exist
* Origin is a remote command endpoint different to the configured, and whose zone is not allowed to access this checkable.

#### event::SetNextCheck <a id="technical-concepts-json-rpc-messages-event-setnextcheck"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SetNextCheck
params    | Dictionary

##### Params

Key         | Type          | Description
------------|---------------|------------------
host        | String        | Host name
service     | String        | Service name
next\_check | Timestamp     | Next scheduled time as UNIX timestamp.

##### Functions

Event Sender: `Checkable::OnNextCheckChanged`
Event Receiver: `NextCheckChangedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### event::SetLastCheckStarted <a id="technical-concepts-json-rpc-messages-event-setlastcheckstarted"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SetLastCheckStarted
params    | Dictionary

##### Params

Key                  | Type      | Description
---------------------|-----------|------------------
host                 | String    | Host name
service              | String    | Service name
last\_check\_started | Timestamp | Last check's start time as UNIX timestamp.

##### Functions

Event Sender: `Checkable::OnLastCheckStartedChanged`
Event Receiver: `LastCheckStartedChangedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### event::SuppressedNotifications <a id="technical-concepts-json-rpc-messages-event-setsupressednotifications"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SuppressedNotifications
params    | Dictionary

##### Params

Key         		 | Type          | Description
-------------------------|---------------|------------------
host        		 | String        | Host name
service     		 | String        | Service name
supressed\_notifications | Number 	 | Bitmask for suppressed notifications.

##### Functions

Event Sender: `Checkable::OnSuppressedNotificationsChanged`
Event Receiver: `SuppressedNotificationsChangedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### event::SetSuppressedNotificationTypes <a id="technical-concepts-json-rpc-messages-event-setsuppressednotificationtypes"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SetSuppressedNotificationTypes
params    | Dictionary

##### Params

Key         		 | Type   | Description
-------------------------|--------|------------------
notification             | String | Notification name
supressed\_notifications | Number | Bitmask for suppressed notifications.

##### Functions

Event Sender: `Notification::OnSuppressedNotificationsChanged`
Event Receiver: `SuppressedNotificationTypesChangedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Notification does not exist.
* Origin endpoint's zone is not allowed to access this notification.


#### event::SetNextNotification <a id="technical-concepts-json-rpc-messages-event-setnextnotification"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SetNextNotification
params    | Dictionary

##### Params

Key                | Type          | Description
-------------------|---------------|------------------
host               | String        | Host name
service            | String        | Service name
notification       | String        | Notification name
next\_notification | Timestamp     | Next scheduled notification time as UNIX timestamp.

##### Functions

Event Sender: `Notification::OnNextNotificationChanged`
Event Receiver: `NextNotificationChangedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Notification does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### event::SetForceNextCheck <a id="technical-concepts-json-rpc-messages-event-setforcenextcheck"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SetForceNextCheck
params    | Dictionary

##### Params

Key       | Type          | Description
----------|---------------|------------------
host      | String        | Host name
service   | String        | Service name
forced    | Boolean       | Forced next check (execute now)

##### Functions

Event Sender: `Checkable::OnForceNextCheckChanged`
Event Receiver: `ForceNextCheckChangedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### event::SetForceNextNotification <a id="technical-concepts-json-rpc-messages-event-setforcenextnotification"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SetForceNextNotification
params    | Dictionary

##### Params

Key       | Type          | Description
----------|---------------|------------------
host      | String        | Host name
service   | String        | Service name
forced    | Boolean       | Forced next check (execute now)

##### Functions

Event Sender: `Checkable::SetForceNextNotification`
Event Receiver: `ForceNextNotificationChangedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### event::SetAcknowledgement <a id="technical-concepts-json-rpc-messages-event-setacknowledgement"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SetAcknowledgement
params    | Dictionary

##### Params

Key        | Type          | Description
-----------|---------------|------------------
host       | String        | Host name
service    | String        | Service name
author     | String        | Acknowledgement author name.
comment    | String        | Acknowledgement comment content.
acktype    | Number        | Acknowledgement type (0=None, 1=Normal, 2=Sticky)
notify     | Boolean       | Notification should be sent.
persistent | Boolean       | Whether the comment is persistent.
expiry     | Timestamp     | Optional expire time as UNIX timestamp.

##### Functions

Event Sender: `Checkable::OnForceNextCheckChanged`
Event Receiver: `ForceNextCheckChangedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### event::ClearAcknowledgement <a id="technical-concepts-json-rpc-messages-event-clearacknowledgement"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::ClearAcknowledgement
params    | Dictionary

##### Params

Key       | Type          | Description
----------|---------------|------------------
host      | String        | Host name
service   | String        | Service name

##### Functions

Event Sender: `Checkable::OnAcknowledgementCleared`
Event Receiver: `AcknowledgementClearedAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### event::SendNotifications <a id="technical-concepts-json-rpc-messages-event-sendnotifications"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::SendNotifications
params    | Dictionary

##### Params

Key       | Type          | Description
----------|---------------|------------------
host      | String        | Host name
service   | String        | Service name
cr        | Serialized CR | Check result
type      | Number        | enum NotificationType, same as `types` for notification objects.
author    | String        | Author name
text      | String        | Notification text

##### Functions

Event Sender: `Checkable::OnNotificationsRequested`
Event Receiver: `SendNotificationsAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone the same as the receiver. This binds notification messages to the HA zone.

#### event::NotificationSentUser <a id="technical-concepts-json-rpc-messages-event-notificationsentuser"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::NotificationSentUser
params    | Dictionary

##### Params

Key           | Type            | Description
--------------|-----------------|------------------
host          | String          | Host name
service       | String          | Service name
notification  | String          | Notification name.
user          | String          | Notified user name.
type          | Number          | enum NotificationType, same as `types` in Notification objects.
cr            | Serialized CR   | Check result.
author        | String          | Notification author (for specific types)
text          | String          | Notification text (for specific types)
command       | String          | Notification command name.

##### Functions

Event Sender: `Checkable::OnNotificationSentToUser`
Event Receiver: `NotificationSentUserAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone the same as the receiver. This binds notification messages to the HA zone.

#### event::NotificationSentToAllUsers <a id="technical-concepts-json-rpc-messages-event-notificationsenttoallusers"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::NotificationSentToAllUsers
params    | Dictionary

##### Params

Key                         | Type            | Description
----------------------------|-----------------|------------------
host                        | String          | Host name
service                     | String          | Service name
notification                | String          | Notification name.
users                       | Array of String | Notified user names.
type                        | Number          | enum NotificationType, same as `types` in Notification objects.
cr                          | Serialized CR   | Check result.
author                      | String          | Notification author (for specific types)
text                        | String          | Notification text (for specific types)
last\_notification          | Timestamp       | Last notification time as UNIX timestamp.
next\_notification          | Timestamp       | Next scheduled notification time as UNIX timestamp.
notification\_number        | Number          | Current notification number in problem state.
last\_problem\_notification | Timestamp       | Last problem notification time as UNIX timestamp.
no\_more\_notifications     | Boolean         | Whether to send future notifications when this notification becomes active on this HA node.

##### Functions

Event Sender: `Checkable::OnNotificationSentToAllUsers`
Event Receiver: `NotificationSentToAllUsersAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone the same as the receiver. This binds notification messages to the HA zone.

#### event::ExecuteCommand <a id="technical-concepts-json-rpc-messages-event-executecommand"></a>

> Location: `clusterevents-check.cpp` and `checkable-check.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::ExecuteCommand
params    | Dictionary

##### Params

Key            | Type          | Description
---------------|---------------|------------------
host           | String        | Host name.
service        | String        | Service name.
command\_type  | String        | `check_command` or `event_command`.
command        | String        | CheckCommand or EventCommand name.
check\_timeout | Number        | Check timeout of the checkable object, if specified as `check_timeout` attribute.
macros         | Dictionary    | Command arguments as key/value pairs for remote execution.
endpoint       | String        | The endpoint to execute the command on.
deadline       | Number        | A Unix timestamp indicating the execution deadline
source         | String        | The execution UUID


##### Functions

**Event Sender:** This gets constructed directly in `Checkable::ExecuteCheck()`, `Checkable::ExecuteEventHandler()` or `ApiActions::ExecuteCommand()` when a remote command endpoint is configured.

* `Get{CheckCommand,EventCommand}()->Execute()` simulates an execution and extracts all command arguments into the `macro` dictionary (inside lib/methods tasks).
* When the endpoint is connected, the message is constructed and sent directly.
* When the endpoint is not connected and not syncing replay logs and 5m after application start, generate an UNKNOWN check result for the user ("not connected").

**Event Receiver:** `ExecuteCommandAPIHandler`

Special handling, calls `ClusterEvents::EnqueueCheck()` for command endpoint checks.
This function enqueues check tasks into a queue which is controlled in `RemoteCheckThreadProc()`.
If the `endpoint` parameter is specified and is not equal to the local endpoint then the message is forwarded to the correct endpoint zone.

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Origin endpoint's zone is not a parent zone of the receiver endpoint.
* `accept_commands = false` in the `api` feature configuration sends back an UNKNOWN check result to the sender.

The receiver constructs a virtual host object and looks for the local CheckCommand object.

Returns UNKNOWN as check result to the sender

* when the CheckCommand object does not exist.
* when there was an exception triggered from check execution, e.g. the plugin binary could not be executed or similar.

The returned messages are synced directly to the sender's endpoint, no cluster broadcast.

> **Note**: EventCommand errors are just logged on the remote endpoint.

### event::UpdateExecutions <a id="technical-concepts-json-rpc-messages-event-updateexecutions"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::UpdateExecutions
params    | Dictionary

##### Params

Key            | Type          | Description
---------------|---------------|------------------
host           | String        | Host name.
service        | String        | Service name.
executions     | Dictionary    | Executions to be updated

##### Functions

**Event Sender:** `ClusterEvents::ExecutedCommandAPIHandler`, `ClusterEvents::UpdateExecutionsAPIHandler`, `ApiActions::ExecuteCommand`
**Event Receiver:** `ClusterEvents::UpdateExecutionsAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

### event::ExecutedCommand <a id="technical-concepts-json-rpc-messages-event-executedcommand"></a>

> Location: `clusterevents.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | event::ExecutedCommand
params    | Dictionary

##### Params

Key            | Type          | Description
---------------|---------------|------------------
host           | String        | Host name.
service        | String        | Service name.
execution      | String        | The execution ID executed.
exitStatus     | Number        | The command exit status.
output         | String        | The command output.
start          | Number        | The unix timestamp at the start of the command execution
end            | Number        | The unix timestamp at the end of the command execution

##### Functions

**Event Sender:** `ClusterEvents::ExecuteCheckFromQueue`, `ClusterEvents::ExecuteCommandAPIHandler`
**Event Receiver:** `ClusterEvents::ExecutedCommandAPIHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Checkable does not exist.
* Origin endpoint's zone is not allowed to access this checkable.

#### config::Update <a id="technical-concepts-json-rpc-messages-config-update"></a>

> Location: `apilistener-filesync.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | config::Update
params    | Dictionary

##### Params

Key        | Type          | Description
-----------|---------------|------------------
update     | Dictionary    | Config file paths and their content.
update\_v2 | Dictionary    | Additional meta config files introduced in 2.4+ for compatibility reasons.

##### Functions

**Event Sender:** `SendConfigUpdate()` called in `ApiListener::SyncClient()` when a new client endpoint connects.
**Event Receiver:** `ConfigUpdateHandler` reads the config update content and stores them in `/var/lib/icinga2/api`.
When it detects a configuration change, the function requests and application restart.

##### Permissions

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* The origin sender is not in a parent zone of the receiver.
* `api` feature does not accept config.

Config updates will be ignored when:

* The zone is not configured on the receiver endpoint.
* The zone is authoritative on this instance (this only happens on a master which has `/etc/icinga2/zones.d` populated, and prevents sync loops)

#### config::UpdateObject <a id="technical-concepts-json-rpc-messages-config-updateobject"></a>

> Location: `apilistener-configsync.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | config::UpdateObject
params    | Dictionary

##### Params

Key                  | Type        | Description
---------------------|-------------|------------------
name                 | String      | Object name.
type                 | String      | Object type name.
version              | Number      | Object version.
config               | String      | Config file content for `_api` packages.
modified\_attributes | Dictionary  | Modified attributes at runtime as key value pairs.
original\_attributes | Array       | Original attributes as array of keys.


##### Functions

**Event Sender:** Either on client connect (full sync), or runtime created/updated object

`ApiListener::SendRuntimeConfigObjects()` gets called when a new endpoint is connected
and runtime created config objects need to be synced. This invokes a call to `UpdateConfigObject()`
to only sync this JsonRpcConnection client.

`ConfigObject::OnActiveChanged` (created or deleted) or `ConfigObject::OnVersionChanged` (updated)
also call `UpdateConfigObject()`.

**Event Receiver:** `ConfigUpdateObjectAPIHandler` calls `ConfigObjectUtility::CreateObject()` in order
to create the object if it is not already existing. Afterwards, all modified attributes are applied
and in case, original attributes are restored. The object version is set as well, keeping it in sync
with the sender.

##### Permissions

###### Sender

Client receiver connects:

The sender only syncs config object updates to a client which can access
the config object, in `ApiListener::SendRuntimeConfigObjects()`.

In addition to that, the client endpoint's zone is checked whether this zone may access
the config object.

Runtime updated object:

Only if the config object belongs to the `_api` package.


###### Receiver

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Origin sender endpoint's zone is in a child zone.
* `api` feature does not accept config
* The received config object type does not exist (this is to prevent failures with older nodes and new object types).

Error handling:

* Log an error if `CreateObject` fails (only if the object does not already exist)
* Local object version is newer than the received version, object will not be updated.
* Compare modified and original attributes and restore any type of change here.


#### config::DeleteObject <a id="technical-concepts-json-rpc-messages-config-deleteobject"></a>

> Location: `apilistener-configsync.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | config::DeleteObject
params    | Dictionary

##### Params

Key                 | Type        | Description
--------------------|-------------|------------------
name                | String      | Object name.
type                | String      | Object type name.
version             | Number      | Object version.

##### Functions

**Event Sender:**

`ConfigObject::OnActiveChanged` (created or deleted) or `ConfigObject::OnVersionChanged` (updated)
call `DeleteConfigObject()`.

**Event Receiver:** `ConfigDeleteObjectAPIHandler`

##### Permissions

###### Sender

Runtime deleted object:

Only if the config object belongs to the `_api` package.

###### Receiver

The receiver will not process messages from not configured endpoints.

Message updates will be dropped when:

* Origin sender endpoint's zone is in a child zone.
* `api` feature does not accept config
* The received config object type does not exist (this is to prevent failures with older nodes and new object types).
* The object in question was not created at runtime, it does not belong to the `_api` package.

Error handling:

* Log an error if `DeleteObject` fails (only if the object does not already exist)

#### pki::RequestCertificate <a id="technical-concepts-json-rpc-messages-pki-requestcertificate"></a>

> Location: `jsonrpcconnection-pki.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | pki::RequestCertificate
params    | Dictionary

##### Params

Key           | Type          | Description
--------------|---------------|------------------
ticket        | String        | Own ticket, or as satellite in CA proxy from local store.
cert\_request | String        | Certificate request content from local store, optional.

##### Functions

Event Sender: `RequestCertificateHandler`
Event Receiver: `RequestCertificateHandler`

##### Permissions

This is an anonymous request, and the number of anonymous clients can be configured
in the `api` feature.

Only valid certificate request messages are processed, and valid signed certificates
won't be signed again.

#### pki::UpdateCertificate <a id="technical-concepts-json-rpc-messages-pki-updatecertificate"></a>

> Location: `jsonrpcconnection-pki.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | pki::UpdateCertificate
params    | Dictionary

##### Params

Key                  | Type          | Description
---------------------|---------------|------------------
status\_code         | Number        | Status code, 0=ok.
cert                 | String        | Signed certificate content.
ca                   | String        | Public CA certificate content.
fingerprint\_request | String        | Certificate fingerprint from the CSR.


##### Functions

**Event Sender:**

* When a client requests a certificate in `RequestCertificateHandler` and the satellite
  already has a signed certificate, the `pki::UpdateCertificate` message is constructed and sent back.
* When the endpoint holding the master's CA private key (and TicketSalt private key) is able to sign
  the request, the `pki::UpdateCertificate` message is constructed and sent back.

**Event Receiver:** `UpdateCertificateHandler`

##### Permissions

Message updates are dropped when

* The origin sender is not in a parent zone of the receiver.
* The certificate fingerprint is in an invalid format.

#### log::SetLogPosition <a id="technical-concepts-json-rpc-messages-log-setlogposition"></a>

> Location: `apilistener.cpp` and `jsonrpcconnection.cpp`

##### Message Body

Key       | Value
----------|---------
jsonrpc   | 2.0
method    | log::SetLogPosition
params    | Dictionary

##### Params

Key                 | Type          | Description
--------------------|---------------|------------------
log\_position       | Timestamp     | The endpoint's log position as UNIX timestamp.


##### Functions

**Event Sender:**

During log replay to a client endpoint in `ApiListener::ReplayLog()`, each processed
file generates a message which updates the log position timestamp.

`ApiListener::ApiTimerHandler()` invokes a check to keep all connected endpoints and
their log position in sync during replay log.

**Event Receiver:** `SetLogPositionHandler`

##### Permissions

The receiver will not process messages from not configured endpoints.
