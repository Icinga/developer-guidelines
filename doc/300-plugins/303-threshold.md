
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
