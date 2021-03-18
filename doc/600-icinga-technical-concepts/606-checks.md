
## Checks<a id="technical-concepts-checks"></a>

### Check Latency and Execution Time <a id="technical-concepts-checks-latency"></a>

Each check command execution logs the start and end time where
Icinga 2 (and the end user) is able to calculate the plugin execution time from it.

```cpp
GetExecutionEnd() - GetExecutionStart()
```

The higher the execution time, the higher the command timeout must be set. Furthermore
users and developers are encouraged to look into plugin optimizations to minimize the
execution time. Sometimes it is better to let an external daemon/script do the checks
and feed them back via REST API.

Icinga 2 stores the scheduled start and end time for a check. If the actual
check execution time differs from the scheduled time, e.g. due to performance
problems or limited execution slots (concurrent checks), this value is stored
and computed from inside the check result.

The difference between the two deltas is called `check latency`.

```cpp
(GetScheduleEnd() - GetScheduleStart()) - CalculateExecutionTime()
```

### Severity <a id="technical-concepts-checks-severity"></a>

The severity attribute is introduced with Icinga v2.11 and provides
a bit mask calculated value from specific checkable object states.

The severity value is pre-calculated for visualization interfaces
such as Icinga Web which sorts the problem dashboard by severity by default.

The higher the severity number is, the more important the problem is.

Flags:

```cpp
/**
 * Severity Flags
 *
 * @ingroup icinga
 */
enum SeverityFlag
{
	SeverityFlagDowntime = 1,
	SeverityFlagAcknowledgement = 2,
	SeverityFlagHostDown = 4,
	SeverityFlagUnhandled = 8,
	SeverityFlagPending = 16,
	SeverityFlagWarning = 32,
	SeverityFlagUnknown = 64,
	SeverityFlagCritical = 128,
};
```


Host:

```cpp
	/* OK/Warning = Up, Critical/Unknown = Down */
	if (!HasBeenChecked())
		severity |= SeverityFlagPending;
	else if (state == ServiceUnknown)
		severity |= SeverityFlagCritical;
	else if (state == ServiceCritical)
		severity |= SeverityFlagCritical;

	if (IsInDowntime())
		severity |= SeverityFlagDowntime;
	else if (IsAcknowledged())
		severity |= SeverityFlagAcknowledgement;
	else
		severity |= SeverityFlagUnhandled;
```


Service:

```cpp
	if (!HasBeenChecked())
		severity |= SeverityFlagPending;
	else if (state == ServiceWarning)
		severity |= SeverityFlagWarning;
	else if (state == ServiceUnknown)
		severity |= SeverityFlagUnknown;
	else if (state == ServiceCritical)
		severity |= SeverityFlagCritical;

	if (IsInDowntime())
		severity |= SeverityFlagDowntime;
	else if (IsAcknowledged())
		severity |= SeverityFlagAcknowledgement;
	else if (m_Host->GetProblem())
		severity |= SeverityFlagHostDown;
	else
		severity |= SeverityFlagUnhandled;
```
