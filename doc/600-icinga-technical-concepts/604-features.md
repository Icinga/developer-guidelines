
## Features <a id="technical-concepts-features"></a>

Features are implemented in specific libraries and can be enabled
using CLI commands.

Features either write specific data or receive data.

Examples for writing data: [DB IDO](14-features.md#db-ido), [Graphite](14-features.md#graphite-carbon-cache-writer), [InfluxDB](14-features.md#influxdb-writer). [GELF](14-features.md#gelfwriter), etc.
Examples for receiving data: [REST API](12-icinga2-api.md#icinga2-api), etc.

The implementation of features makes use of existing libraries
and functionality. This makes the code more abstract, but shorter
and easier to read.

Features register callback functions on specific events they want
to handle. For example the `GraphiteWriter` feature subscribes to
new CheckResult events.

Each time Icinga 2 receives and processes a new check result, this
event is triggered and forwarded to all subscribers.

The GraphiteWriter feature calls the registered function and processes
the received data. Features which connect Icinga 2 to external interfaces
normally parse and reformat the received data into an applicable format.

Since this check result signal is blocking, many of the features include a work queue
with asynchronous task handling.

The GraphiteWriter uses a TCP socket to communicate with the carbon cache
daemon of Graphite. The InfluxDBWriter is instead writing bulk metric messages
to InfluxDB's HTTP API, similar to Elasticsearch.
