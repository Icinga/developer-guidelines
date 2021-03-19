The check scheduler starts a thread which loops forever. It waits for
check events being inserted into `m_IdleCheckables`.

If the current pending check event number is larger than the configured
max concurrent checks, the thread waits up until it there's slots again.

In addition, further checks on enabled checks, check periods, etc. are
performed. Once all conditions have passed, the next check timestamp is
calculated and updated. This also is the timestamp where Icinga expects
a new check result ("freshness check").

The object is removed from idle checkables, and inserted into the
pending checkables list. This can be seen via REST API metrics for the
checker component feature as well.

The actual check execution happens asynchronously using the application's
thread pool.

Once the check returns, it is removed from pending checkables and again
inserted into idle checkables. This ensures that the scheduler takes this
checkable event into account in the next iteration.

### Start <a id="technical-concepts-check-scheduler-start"></a>

When checkable objects get activated during the startup phase,
the checker feature registers a handler for this event. This is due
to the fact that the `checker` feature is fully optional, and e.g. not
used on command endpoint clients.

Whenever such an object activation signal is triggered, Icinga 2 checks
whether it is [authoritative for this object](19-technical-concepts.md#technical-concepts-cluster-ha-object-authority).
This means that inside an HA enabled zone with two endpoints, only non-paused checkable objects are
actively inserted into the idle checkable list for the check scheduler.

### Initial Check <a id="technical-concepts-check-scheduler-initial"></a>

When a new checkable object (host or service) is initially added to the
configuration, Icinga 2 performs the following during startup:

* `Checkable::Start()` is called and calculates the first check time
* With a spread delta, the next check time is actually set.

If the next check should happen within a time frame of 60 seconds,
Icinga 2 calculates a delta from a random value. The minimum of `check_interval`
and 60 seconds is used as basis, multiplied with a random value between 0 and 1.

In the best case, this check gets immediately executed after application start.
The worst case scenario is that the check is scheduled 60 seconds after start
the latest.

The reasons for delaying and spreading checks during startup is that
the application typically needs more resources at this time (cluster connections,
feature warmup, initial syncs, etc.). Immediate check execution with
thousands of checks could lead into performance problems, and additional
events for each received check results.

Therefore the initial check window is 60 seconds on application startup,
random seed for all checkables. This is not predictable over multiple restarts
for specific checkable objects, the delta changes every time.

### Scheduling Offset <a id="technical-concepts-check-scheduler-offset"></a>

There's a high chance that many checkable objects get executed at the same time
and interval after startup. The initial scheduling spreads that a little, but
Icinga 2 also attempts to ensure to keep fixed intervals, even with high check latency.

During startup, Icinga 2 calculates the scheduling offset from a random number:

* `Checkable::Checkable()` calls `SetSchedulingOffset()` with `Utility::Random()`
* The offset is a pseudo-random integral value between `0` and `RAND_MAX`.

Whenever the next check time is updated with `Checkable::UpdateNextCheck()`,
the scheduling offset is taken into account.

Depending on the state type (SOFT or HARD), either the `retry_interval` or `check_interval`
is used. If the interval is greater than 1 second, the time adjustment is calculated in the
following way:

`now * 100 + offset` divided by `interval * 100`, using the remainder (that's what `fmod()` is for)
and dividing this again onto base 100.

Example: offset is 6500, interval 300, now is 1542190472.

```
1542190472 * 100 + 6500 = 154219053714
300 * 100 = 30000
154219053714 / 30000 = 5140635.1238

(5140635.1238 - 5140635.0) * 30000 = 3714
3714 / 100 = 37.14
```

37.15 seconds as an offset would be far too much, so this is again used as a calculation divider for the
real offset with the base of 5 times the actual interval.

Again, the remainder is calculated from the offset and `interval * 5`. This is divided onto base 100 again,
with an additional 0.5 seconds delay.

Example: offset is 6500, interval 300.

```
6500 / 300 = 21.666666666666667
(21.666666666666667 - 21.0) * 300 = 200
200 / 100 = 2
2 + 0.5 = 2.5
```

The minimum value between the first adjustment and the second offset calculation based on the interval is
taken, in the above example `2.5` wins.

The actual next check time substracts the adjusted time from the future interval addition to provide
a more widespread scheduling time among all checkable objects.

`nextCheck = now - adj + interval`

You may ask, what other values can happen with this offset calculation. Consider calculating more examples
with different interval settings.

Example: offset is 34567, interval 60, now is 1542190472.

```
1542190472 * 100 + 34567 = 154219081767
60 * 100 = 6000
154219081767 / 6000 = 25703180.2945
(25703180.2945 - 25703180.0) * 6000 / 100 = 17.67

34567 / 60 = 576.116666666666667
(576.116666666666667 - 576.0) * 60 / 100 + 0.5 = 1.2
```

`1m` interval starts at `now + 1.2s`.

Example: offset is 12345, interval 86400, now is 1542190472.

```
1542190472 * 100 + 12345 = 154219059545
86400 * 100 = 8640000
154219059545 / 8640000 = 17849.428188078703704
(17849.428188078703704 - 17849) * 8640000 = 3699545
3699545 / 100 = 36995.45

12345 / 86400 = 0.142881944444444
0.142881944444444 * 86400 / 100 + 0.5 = 123.95
```

`1d` interval starts at `now + 2m4s`.

> **Note**
>
> In case you have a better algorithm at hand, feel free to discuss this in a PR on GitHub.
> It needs to fulfill two things: 1) spread and shuffle execution times on each `next_check` update
> 2) not too narrowed window for both long and short intervals
     > Application startup and initial checks need to be handled with care in a slightly different
     > fashion.

When `SetNextCheck()` is called, there are signals registered. One of them sits
inside the `CheckerComponent` class whose handler `CheckerComponent::NextCheckChangedHandler()`
deletes/inserts the next check event from the scheduling queue. This basically
is a list with multiple indexes with the keys for scheduling info and the object.
