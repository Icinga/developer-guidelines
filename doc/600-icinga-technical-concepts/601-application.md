### CLI Commands <a id="technical-concepts-application-cli-commands"></a>

The Icinga 2 application is managed with different CLI sub commands.
`daemon` takes care about loading the configuration files, running the
application as daemon, etc.
Other sub commands allow to enable features, generate and request
TLS certificates or enter the debug console.

The main entry point for each CLI command parses the command line
parameters and then triggers the required actions.

### daemon CLI command <a id="technical-concepts-application-cli-commands-daemon"></a>

This CLI command loads the configuration files, starting with `icinga2.conf`.
The [configuration compiler](19-technical-concepts.md#technical-concepts-configuration) parses the
file and detects additional file includes, constants, and any other DSL
specific declaration.

At this stage, the configuration will already be checked against the
defined grammar in the scanner, and custom object validators will also be
checked.

If the user provided `-C/--validate`, the CLI command returns with the
validation exit code.

When running as daemon, additional parameters are checked, e.g. whether
this application was triggered by a reload, needs to daemonize with fork()
involved and update the object's authority. The latter is important for
HA-enabled cluster zones.
