
## Core <a id="technical-concepts-core"></a>

### Core: Reload Handling <a id="technical-concepts-core-reload"></a>

The initial design of the reload state machine looks like this:

* receive reload signal SIGHUP
* fork a child process, start configuration validation in parallel work queues
* parent process continues with old configuration objects and the event scheduling
  (doing checks, replicating cluster events, triggering alert notifications, etc.)
* validation NOT ok: child process terminates, parent process continues with old configuration state
* validation ok: child process signals parent process to terminate and save its current state (all events until now) into the icinga2 state file
* parent process shuts down writing icinga2.state file
* child process waits for parent process gone, reads the icinga2 state file and synchronizes all historical and status data
* child becomes the new session leader

Since Icinga 2.6, there are two processes when checked with `ps aux | grep icinga2` or `pidof icinga2`.
This was to ensure that feature file descriptors don't leak into the plugin process (e.g. DB IDO MySQL sockets).

Icinga 2.9 changed the reload handling a bit with SIGUSR2 signals
and systemd notifies.

With systemd, it could occur that the tree was broken thus resulting
in killing all remaining processes on stop, instead of a clean exit.
You can read the full story [here](https://github.com/Icinga/icinga2/issues/7309).

With 2.11 you'll now see 3 processes:

- The umbrella process which takes care about signal handling and process spawning/stopping
- The main process with the check scheduler, notifications, etc.
- The execution helper process

During reload, the umbrella process spawns a new reload process which validates the configuration.
Once successful, the new reload process signals the umbrella process that it is finished.
The umbrella process forwards the signal and tells the old main process to shutdown.
The old main process writes the icinga2.state file. The umbrella process signals
the reload process that the main process terminated.

The reload process was in idle wait before, and now continues to read the written
state file and run the event loop (checks, notifications, "events", ...). The reload
process itself also spawns the execution helper process again.
