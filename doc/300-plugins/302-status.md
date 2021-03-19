
Value | Status    | Description
------|-----------|-------------------------------
0     | OK        | The check went fine and everything is considered working.
1     | Warning   | The check is above the given warning threshold, or anything else is suspicious requiring attention before it breaks.
2     | Critical  | The check exceeded the critical threshold, or something really is broken and will harm the production environment.
3     | Unknown   | Invalid parameters, low level resource errors (IO device busy, no fork resources, TCP sockets, etc.) preventing the actual check. Higher level errors such as DNS resolving, TCP connection timeouts should be treated as `Critical` instead. Whenever the plugin reaches its timeout (best practice) it should also terminate with `Unknown`.

Keep in mind that these are service states. Icinga automatically maps
the [host state](03-monitoring-basics.md#check-result-state-mapping) from the returned plugin states.
