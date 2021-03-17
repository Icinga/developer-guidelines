
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
