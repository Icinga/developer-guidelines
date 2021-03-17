
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
