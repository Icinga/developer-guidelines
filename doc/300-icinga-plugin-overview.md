// todo: add info for powershell -> 08-icinga-for-windows.md

### Plugin API <a id="service-monitoring-plugin-api"></a>

Icinga 2 supports the native plugin API specification from the Monitoring Plugins project.
It is defined in the [Monitoring Plugins](https://www.monitoring-plugins.org) guidelines.

The Icinga documentation revamps the specification into our
own guideline enriched with examples and best practices.

#### Output <a id="service-monitoring-plugin-api-output"></a>

The output should be as short and as detailed as possible. The
most common cases include:

- Viewing a problem list in Icinga Web and dashboards
- Getting paged about a problem
- Receiving the alert on the CLI or forwarding it to external (ticket) systems

Examples:

```
<STATUS>: <A short description what happened>

OK: MySQL connection time is fine (0.0002s)
WARNING: MySQL connection time is slow (0.5s > 0.1s threshold)
CRITICAL: MySQL connection time is causing degraded performance (3s > 0.5s threshold)
```

Icinga supports reading multi-line output where Icinga Web
only shows the first line in the listings and everything in the detail view.

Example for an end2end check with many smaller test cases integrated:

```
OK: Online banking works.
Testcase 1: Site reached.
Testcase 2: Attempted login, JS loads.
Testcase 3: Login succeeded.
Testcase 4: View current state works.
Testcase 5: Transactions fine.
```

If the extended output shouldn't be visible in your monitoring, but only for testing,
it is recommended to implement the `--verbose` plugin parameter to allow
developers and users to debug further. Check [here](05-service-monitoring.md#service-monitoring-plugin-api-verbose)
for more implementation tips.

> **Tip**
>
> More debug output also helps when implementing your plugin.
>
> Best practice is to have the plugin parameter and handling implemented first,
> then add it anywhere you want to see more, e.g. from initial database connections
> to actual query results.


#### Status <a id="service-monitoring-plugin-api-status"></a>

Value | Status    | Description
------|-----------|-------------------------------
0     | OK        | The check went fine and everything is considered working.
1     | Warning   | The check is above the given warning threshold, or anything else is suspicious requiring attention before it breaks.
2     | Critical  | The check exceeded the critical threshold, or something really is broken and will harm the production environment.
3     | Unknown   | Invalid parameters, low level resource errors (IO device busy, no fork resources, TCP sockets, etc.) preventing the actual check. Higher level errors such as DNS resolving, TCP connection timeouts should be treated as `Critical` instead. Whenever the plugin reaches its timeout (best practice) it should also terminate with `Unknown`.

Keep in mind that these are service states. Icinga automatically maps
the [host state](03-monitoring-basics.md#check-result-state-mapping) from the returned plugin states.

#### Thresholds <a id="service-monitoring-plugin-api-thresholds"></a>

A plugin calculates specific values and may decide about the exit state on its own.
This is done with thresholds - warning and critical values which are compared with
the actual value. Upon this logic, the exit state is determined.

Imagine the following value and defined thresholds:

```
ptc_value = 57.8

warning = 50
critical = 60
```

Whenever `ptc_value` is higher than warning or critical, it should return
the appropriate [state](05-service-monitoring.md#service-monitoring-plugin-api-status).

The threshold evaluation order also is important:

* Critical thresholds are evaluated first and superseed everything else.
* Warning thresholds are evaluated second
* If no threshold is matched, return the OK state

Avoid using hardcoded threshold values in your plugins, always
add them to the argument parser.

Example for Python:

```python
import argparse
import signal
import sys

if __name__ == '__main__':
    parser = argparse.ArgumentParser()

    parser.add_argument("-w", "--warning", help="Warning threshold. Single value or range, e.g. '20:50'.")
    parser.add_argument("-c", "--critical", help="Critical threshold. Single vluae or range, e.g. '25:45'.")

    args = parser.parse_args()
```

Users might call plugins only with the critical threshold parameter,
leaving out the warning parameter. Keep this in mind when evaluating
the thresholds, always check if the parameters have been defined before.

```python
    if args.critical:
        if ptc_value > args.critical:
            print("CRITICAL - ...")
            sys.exit(2) # Critical

    if args.warning:
        if ptc_value > args.warning:
            print("WARNING - ...")
            sys.exit(1) # Warning

    print("OK - ...")
    sys.exit(0) # OK
```

The above is a simplified example for printing the [output](05-service-monitoring.md#service-monitoring-plugin-api-output)
and using the [state](05-service-monitoring.md#service-monitoring-plugin-api-status)
as exit code.

Before diving into the implementation, learn more about required
[performance data metrics](05-service-monitoring.md#service-monitoring-plugin-api-performance-data-metrics)
and more best practices below.

##### Threshold Ranges <a id="service-monitoring-plugin-api-thresholds-ranges"></a>

Threshold ranges can be used to specify an alert window, e.g. whenever a calculated
value is between a lower and higher critical threshold.

The schema for threshold ranges looks as follows. The `@` character in square brackets
is optional.

```
[@]start:end
```

There are a few requirements for ranges:

* `start <= end`. Add a check in your code and let the user know about problematic values.

```
10:20 	# OK

30:10 	# Error
```

* `start:` can be omitted if its value is 0. This is the default handling for single threshold values too.

```
10 	# Every value > 10 and < 0, outside of 0..10
```

* If `end` is omitted, assume end is infinity.

```
10: 	# < 10, outside of 10..∞
```

* In order to specify negative infinity, use the `~` character.

```
~:10	# > 10, outside of -∞..10
```

* Raise alert if value is outside of the defined range.

```
10:20 	# < 10 or > 20, outside of 10..20
```

* Start with `@` to raise an alert if the value is **inside** the defined range, inclusive start/end values.

```
@10:20	# >= 10 and <= 20, inside of 10..20
```

Best practice is to either implement single threshold values, or fully support ranges.
This requires parsing the input parameter values, therefore look for existing libraries
already providing this functionality.

[check_tinkerforge](https://github.com/NETWAYS/check_tinkerforge/blob/master/check_tinkerforge.py)
implements a simple parser to avoid dependencies.


#### Performance Data Metrics <a id="service-monitoring-plugin-api-performance-data-metrics"></a>

Performance data metrics must be appended to the plugin output with a preceding `|` character.
The schema is as follows:

```
<output> | 'label'=value[UOM];[warn];[crit];[min];[max]
```

The label should be encapsulated with single quotes. Avoid spaces or special characters such
as `%` in there, this could lead to problems with metric receivers such as Graphite.

Labels must not include `'` and `=` characters. Keep the label length as short and unique as possible.

Example:

```
'load1'=4.7
```

Values must respect the C/POSIX locale and not implement e.g. German locale for floating point numbers with `,`.
Icinga sets `LC_NUMERIC=C` to enforce this locale on plugin execution.

##### Unit of Measurement (UOM) <a id="service-monitoring-plugin-api-performance-data-metrics-uom"></a>

```
'rta'=12.445000ms 'pl'=0%
```

Icinga interprets the plugins' UoMs as follows:

* If the UoM is "c", the value is a continuous counter (e.g. interface traffic counters).
* Otherwise if the UoM is listed in the table below (case-insensitive if possible),
  Icinga normalizes the value (and warn, crit, min, max) to the respective common base.
  That common base is also used by the metric writers (if any).

Common base        | UoMs
-------------------|---------------------------------------
bytes              | B, KB, MB, ..., YB, KiB, MiB, ..., YiB
bits               | b, kb, mb, ..., yb, kib, mib, ..., yib
packets            | packets
seconds            | ns, us, ms, s, m, h, d
percent            | %
amperes            | nA, uA, mA, A, kA, MA, GA, ..., YA
ohms               | nO, uO, mO, O, kO, MO, GO, ..., YO
volts              | nV, uV, mV, V, kV, MV, GV, ..., YV
watts              | nW, uW, mW, W, kW, MW, GW, ..., YW
ampere-seconds     | nAs, uAs, mAs, As, kAs, MAs, GAs, ..., YAs, all of these also for Am and Ah
watt-hours         | nWh, uWh, mWh, Wh, kWh, MWh, GWh, ..., YWh, all of these also for Wm and Ws
lumens             | lm
decibel-milliwatts | dBm
grams              | ng, ug, mg, g, kg, t
degrees-celsius    | C
degrees-fahrenheit | F
degrees-kelvin     | K
liters             | ml, l, hl

Some plugins change the UoM for different sizing, e.g. returning the disk usage in MB and later GB
for the same performance data label. This is to ensure that graphs always look the same.

* Otherwise the UoM is discarted (as if none was given).

A value without any UoM may be an integer or floating point number
for any type (processes, users, etc.).

##### Thresholds and Min/Max <a id="service-monitoring-plugin-api-performance-data-metrics-thresholds-min-max"></a>

Next to the performance data value, warn, crit, min, max can optionally be provided. They must be separated
with the semi-colon `;` character. They share the same UOM with the performance data value.

```
$ check_ping -4 -H icinga.com -c '200,15%' -w '100,5%'

PING OK - Packet loss = 0%, RTA = 12.44 ms|rta=12.445000ms;100.000000;200.000000;0.000000 pl=0%;5;15;0
```

##### Multiple Performance Data Values <a id="service-monitoring-plugin-api-performance-data-metrics-multiple"></a>

Multiple performance data values must be joined with a space character. The below example
is from the [check_load](10-icinga-template-library.md#plugin-check-command-load) plugin.

```
load1=4.680;1.000;2.000;0; load5=0.000;5.000;10.000;0; load15=0.000;10.000;20.000;0;
```

#### Timeout <a id="service-monitoring-plugin-api-timeout"></a>

Icinga has a safety mechanism where it kills processes running for too
long. The timeout can be specified in [CheckCommand objects](09-object-types.md#objecttype-checkcommand)
or on the host/service object.

Best practice is to control the timeout in the plugin itself
and provide a clear message followed by the Unknown state.

Example in Python taken from [check_tinkerforge](https://github.com/NETWAYS/check_tinkerforge/blob/master/check_tinkerforge.py):

```python
import argparse
import signal
import sys

def handle_sigalrm(signum, frame, timeout=None):
    output('Plugin timed out after %d seconds' % timeout, 3)

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    # ... add more arguments
    parser.add_argument("-t", "--timeout", help="Timeout in seconds (default 10s)", type=int, default=10)
    args = parser.parse_args()

    signal.signal(signal.SIGALRM, partial(handle_sigalrm, timeout=args.timeout))
    signal.alarm(args.timeout)

    # ... perform the check and generate output/status
```

#### Versions <a id="service-monitoring-plugin-api-versions"></a>

Plugins should provide a version via `-V` or `--version` parameter
which is bumped on releases. This allows to identify problems with
too old or new versions on the community support channels.

Example in Python taken from [check_tinkerforge](https://github.com/NETWAYS/check_tinkerforge/blob/master/check_tinkerforge.py):

```python
import argparse
import signal
import sys

__version__ = '0.9.1'

if __name__ == '__main__':
    parser = argparse.ArgumentParser()

    parser.add_argument('-V', '--version', action='version', version='%(prog)s v' + sys.modules[__name__].__version__)
```

#### Verbose <a id="service-monitoring-plugin-api-verbose"></a>

Plugins should provide a verbose mode with `-v` or `--verbose` in order
to show more detailed log messages. This helps to debug and analyse the
flow and execution steps inside the plugin.

Ensure to add the parameter prior to implementing the check logic into
the plugin.

Example in Python taken from [check_tinkerforge](https://github.com/NETWAYS/check_tinkerforge/blob/master/check_tinkerforge.py):

```python
import argparse
import signal
import sys

if __name__ == '__main__':
    parser = argparse.ArgumentParser()

    parser.add_argument('-v', '--verbose', action='store_true')

    if args.verbose:
        print("Verbose debug output")
```


### Create a new Plugin <a id="service-monitoring-plugin-new"></a>

Sometimes an existing plugin does not satisfy your requirements. You
can either kindly contact the original author about plans to add changes
and/or create a patch.

If you just want to format the output and state of an existing plugin
it might also be helpful to write a wrapper script. This script
could pass all configured parameters, call the plugin script, parse
its output/exit code and return your specified output/exit code.

On the other hand plugins for specific services and hardware might not yet
exist.

> **Tip**
>
> Watch this presentation from Icinga Camp Berlin to learn more
> about [How to write checks that don't suck](https://www.youtube.com/watch?v=Ey_APqSCoFQ).

Common best practices:

* Choose the programming language wisely
* Scripting languages (Bash, Python, Perl, Ruby, PHP, etc.) are easier to write and setup but their check execution might take longer (invoking the script interpreter as overhead, etc.).
* Plugins written in C/C++, Go, etc. improve check execution time but may generate an overhead with installation and packaging.
* Use a modern VCS such as Git for developing the plugin, e.g. share your plugin on GitHub and let it sync to [Icinga Exchange](https://exchange.icinga.com).
* **Look into existing plugins endorsed by community members.**

Implementation hints:

* Add parameters with key-value pairs to your plugin. They should allow long names (e.g. `--host localhost`) and also short parameters (e.g. `-H localhost`)
* `-h|--help` should print the version and all details about parameters and runtime invocation. Note: Python's ArgParse class provides this OOTB.
* `--version` should print the plugin [version](05-service-monitoring.md#service-monitoring-plugin-api-versions).
* Add a [verbose/debug output](05-service-monitoring.md#service-monitoring-plugin-api-verbose) functionality for detailed on-demand logging.
* Respect the exit codes required by the [Plugin API](05-service-monitoring.md#service-monitoring-plugin-api).
* Always add [performance data](05-service-monitoring.md#service-monitoring-plugin-api-performance-data-metrics) to your plugin output.
* Allow to specify [warning/critical thresholds](05-service-monitoring.md#service-monitoring-plugin-api-thresholds) as parameters.

Example skeleton:

```
# 1. include optional libraries
# 2. global variables
# 3. helper functions and/or classes
# 4. define timeout condition

if (<timeout_reached>) then
  print "UNKNOWN - Timeout (...) reached | 'time'=30.0
endif

# 5. main method

<execute and fetch data>

if (<threshold_critical_condition>) then
  print "CRITICAL - ... | 'time'=0.1 'myperfdatavalue'=5.0
  exit(2)
else if (<threshold_warning_condition>) then
  print "WARNING - ... | 'time'=0.1 'myperfdatavalue'=3.0
  exit(1)
else
  print "OK - ... | 'time'=0.2 'myperfdatavalue'=1.0
endif
```

There are various plugin libraries available which will help
with plugin execution and output formatting too, for example
[nagiosplugin from Python](https://pypi.python.org/pypi/nagiosplugin/).

> **Note**
>
> Ensure to test your plugin properly with special cases before putting it
> into production!

Once you've finished your plugin please upload/sync it to [Icinga Exchange](https://exchange.icinga.com/new).
Thanks in advance!

