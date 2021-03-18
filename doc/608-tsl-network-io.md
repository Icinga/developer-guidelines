
## TLS Network IO <a id="technical-concepts-tls-network-io"></a>

### TLS Connection Handling <a id="technical-concepts-tls-network-io-connection-handling"></a>

Icinga supports two connection directions, controlled via the `host` attribute
inside the Endpoint objects:

* Outgoing connection attempts
* Incoming connection handling

Once the connection is established, higher layers can exchange JSON-RPC and
HTTP messages. It doesn't matter which direction these message go.

This offers a big advantage over single direction connections, just like
polling via HTTP only. Also, connections are kept alive as long as data
is transmitted.

When the master connects to the child zone member(s), this requires more
resources there. Keep this in mind when endpoints are not reachable, the
TCP timeout blocks other resources. Moving a satellite zone in the middle
between masters and agents helps to split the tasks - the master
processes and stores data, deploys configuration and serves the API. The
satellites schedule the checks, connect to the agents and receive
check results.

Agents/Clients can also connect to the parent endpoints - be it a master or
a satellite. This is the preferred way out of a DMZ, and also reduces the
overhead with connecting to e.g. 2000 agents on the master. You can
benchmark this when TCP connections are broken and timeouts are encountered.

#### Master Processes Incoming Connection <a id="technical-concepts-tls-network-io-connection-handling-incoming"></a>

* The node starts a new ApiListener, this invokes `AddListener()`
    * Setup TLS Context (SslContext)
    * Initialize global I/O engine and create a TCP acceptor
    * Resolve bind host/port (optional)
    * Listen on IPv4 and IPv6
    * Re-use socket address and port
    * Listen on port 5665 with `INT_MAX` possible sockets
* Spawn a new Coroutine which listens for new incoming connections as 'TCP server' pattern
    * Accept new connections asynchronously
    * Spawn a new Coroutine which handles the new client connection in a different context, Role: Server

#### Master Connects Outgoing <a id="technical-concepts-tls-network-io-connection-handling-outgoing"></a>

* The node starts a timer in a 10 seconds interval with `ApiReconnectTimerHandler()` as callback
    * Loop over all configured zones, exclude global zones and not direct parent/child zones
    * Get the endpoints configured in the zones, exclude: local endpoint, no 'host' attribute, already connected or in progress
    * Call `AddConnection()`
* Spawn a new Coroutine after making the TLS context
    * Use the global I/O engine for socket I/O
    * Create TLS stream
    * Connect to endpoint host/port details
    * Handle the client connection, Role: Client

#### TLS Handshake <a id="technical-concepts-tls-network-io-connection-handling-handshake"></a>

* Create a TLS connection in sslConn and perform an asynchronous TLS handshake
* Get the peer certificate
* Verify the presented certificate: `ssl::verify_peer` and `ssl::verify_client_once`
* Get the certificate CN and compare it against the endpoint name - if not matching, return and close the connection

#### Data Exchange <a id="technical-concepts-tls-network-io-connection-data-exchange"></a>

Everything runs through TLS, we don't use any "raw" connections nor plain message handling.

HTTP and JSON-RPC messages share the same port and API, so additional handling is required.

On a new connection and successful TLS handshake, the first byte is read. This either
is a JSON-RPC message in Netstring format starting with a number, or plain HTTP.

```
HTTP/1.1

2:{}
```

Depending on this, `ClientJsonRpc` or `ClientHttp` are assigned.

JSON-RPC:

* Create a new JsonRpcConnection object
    * When the endpoint object is configured, spawn a Coroutine which takes care of syncing the client (file and runtime config, replay log, etc.)
    * No endpoint treats this connection as anonymous client, with a configurable limit. This client may send a CSR signing request for example.
    * Start the JsonRpcConnection - this spawns Coroutines to HandleIncomingMessages, WriteOutgoingMessages, HandleAndWriteHeartbeats and CheckLiveness

HTTP:

* Create a new HttpServerConnection
    * Start the HttpServerConnection - this spawns Coroutines to ProcessMessages and CheckLiveness


All the mentioned Coroutines run asynchronously using the global I/O engine's context.
More details on this topic can be found in [this blogpost](https://www.netways.de/blog/2019/04/04/modern-c-programming-coroutines-with-boost/).

The lower levels of context switching and sharing or event polling are
hidden in Boost ASIO, Beast, Coroutine and Context libraries.

#### Data Exchange: Coroutines and I/O Engine <a id="technical-concepts-tls-network-io-connection-data-exchange-coroutines"></a>

Light-weight and fast operations such as connection handling or TLS handshakes
are performed in the default `IoBoundWorkSlot` pool inside the I/O engine.

The I/O engine has another pool available: `CpuBoundWork`.

This is used for processing CPU intensive tasks, such as handling a HTTP request.
Depending on the available CPU cores, this is limited to `std::thread::hardware_concurrency() * 3u / 2u`.

```
1 core * 3 / 2 = 1
2 cores * 3 / 2 = 3
8 cores * 3 / 2 = 12
16 cores * 3 / 2 = 24
```

The I/O engine itself is used with all network I/O in Icinga, not only the cluster
and the REST API. Features such as Graphite, InfluxDB, etc. also consume its functionality.

There are 2 * CPU cores threads available which run the event loop
in the I/O engine. This polls the I/O service with `m_IoService.run();`
and triggers an asynchronous event progress for waiting coroutines.

<!--
## REST API <a id="technical-concepts-rest-api"></a>

Icinga 2 provides its own HTTP server which shares the port 5665 with
the JSON-RPC cluster protocol.
-->
