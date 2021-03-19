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

