This chapter targets all users who have been asked by developers to provide
a stack trace or coredump if the application crashed. It is also useful
for developers working with different debuggers.

> **Note:**
>
> This is intentionally mentioned before any development insights
> as debugging is a more frequent and commonly asked question.

### Debug Requirements <a id="debug-requirements"></a>

Make sure that the debug symbols are available for Icinga 2.
The Icinga 2 packages provide a debug package which must be
installed separately for all involved binaries, like `icinga2-bin`
or `icinga2-ido-mysql`.

Distribution       | Command
-------------------|------------------------------------------
Debian/Ubuntu      | `apt-get install icinga2-dbg`
RHEL/CentOS        | `yum install icinga2-debuginfo`
Fedora             | `dnf install icinga2-debuginfo icinga2-bin-debuginfo icinga2-ido-mysql-debuginfo`
SLES/openSUSE      | `zypper install icinga2-bin-debuginfo icinga2-ido-mysql-debuginfo`

Furthermore, you may also have to install debug symbols for Boost and your C++ library.

If you're building your own binaries, you should use the `-DCMAKE_BUILD_TYPE=Debug` cmake
build flag for debug builds.


### GDB as Debugger <a id="development-debug-gdb"></a>

Install GDB in your development environment.

Distribution       | Command
-------------------|------------------------------------------
Debian/Ubuntu      | `apt-get install gdb`
RHEL/CentOS        | `yum install gdb`
Fedora             | `dnf install gdb`
SLES/openSUSE      | `zypper install gdb`

#### GDB Run <a id="development-debug-gdb-run"></a>

Run the icinga2 binary `/usr/lib{,64}/icinga2/sbin/icinga2` with gdb, `/usr/bin/icinga2` is a shell wrapper.

```
gdb --args /usr/lib/icinga2/sbin/icinga2 daemon

(gdb) set follow-fork-mode child
```

When gdb halts on SIGUSR2, press `c` to continue. This signal originates from the umbrella
process and can safely be ignored.


> **Note**
>
> Since v2.11 we would attach to the umbrella process spawned with `/usr/lib/icinga2/sbin/icinga2`,
> therefore rather attach to a running process.
>
```bash
# Typically the order of PIDs is: 1) umbrella 2) spawn helper 3) main process
pidof icinga2

gdb -p $(pidof icinga2 | cut -d ' ' -f3)
```

> **Note**
>
> If gdb tells you it's missing debug symbols, quit gdb and install
> them: `Missing separate debuginfos, use: debuginfo-install ...`

Run/restart the application.

```
(gdb) r
```

Kill the running application.

```
(gdb) k
```

Continue after breakpoint.

```
(gdb) c
```

#### GDB Core Dump <a id="development-debug-gdb-coredump"></a>

Either attach to the running process using `gdb -p PID` or start
a new gdb run.

```
(gdb) r
(gdb) generate-core-file
```

#### GDB Backtrace <a id="development-debug-gdb-backtrace"></a>

If Icinga 2 aborted its operation abnormally, generate a backtrace.

> **Note**
>
> Please install the [required debug symbols](21-development.md#debug-requirements)
> prior to generating a backtrace.

`thread apply all` is important here since this includes all running threads.
We need this information when e.g. debugging dead locks and hanging features.

```
(gdb) bt
(gdb) thread apply all bt full
```

If gdb stops at a SIGPIPE signal please disable the signal before
running Icinga 2. This isn't an error, but we need to workaround it.

```
(gdb) handle SIGPIPE nostop noprint pass
(gdb) r
```

If you create a [new issue](https://github.com/Icinga/icinga2/issues),
make sure to attach as much detail as possible.

#### GDB Backtrace from Running Process <a id="development-debug-gdb-backtrace-running"></a>

If Icinga 2 is still running, generate a full backtrace from the running
process and store it into a new file (e.g. for debugging dead locks).

> **Note**
>
> Please install the [required debug symbols](21-development.md#debug-requirements)
> prior to generating a backtrace.

Icinga 2 runs with 2 processes: main and command executor, therefore generate two backtrace logs
and add them to the GitHub issue.

```bash
for pid in $(pidof icinga2); do gdb -p $pid -batch -ex "thread apply all bt full" -ex "detach" -ex "q" > gdb_bt_${pid}_`date +%s`.log; done
```

#### GDB Thread List from Running Process <a id="development-debug-gdb-thread-list-running"></a>

Instead of a full backtrace, you sometimes just need a list of running threads.

```bash
for pid in $(pidof icinga2); do gdb -p $pid -batch -ex "info threads" -ex "detach" -ex "q" > gdb_threads_${pid}_`date +%s`.log; done
```

#### GDB Backtrace Stepping <a id="development-debug-gdb-backtrace-stepping"></a>

Identifying the problem may require stepping into the backtrace, analysing
the current scope, attributes, and possible unmet requirements. `p` prints
the value of the selected variable or function call result.

```
(gdb) up
(gdb) down
(gdb) p checkable
(gdb) p checkable.px->m_Name
```

#### GDB Breakpoints <a id="development-debug-gdb-breakpoint"></a>

To set a breakpoint to a specific function call, or file specific line.

```
(gdb) b checkable.cpp:125
(gdb) b icinga::Checkable::SetEnablePerfdata
```

GDB will ask about loading the required symbols later, select `yes` instead
of `no`.

Then run Icinga 2 until it reaches the first breakpoint. Continue with `c`
afterwards.

```
(gdb) run
(gdb) c
```

In case you want to step into the next line of code, use `n`. If there is a
function call where you want to step into, use `s`.

```
(gdb) n

(gdb) s
```

If you want to delete all breakpoints, use `d` and select `yes`.

```
(gdb) d
```

> **Tip**
>
> When debugging exceptions, set your breakpoint like this: `b __cxa_throw`.

Breakpoint Example:

```
(gdb) b __cxa_throw
(gdb) r
(gdb) up
....
(gdb) up
#11 0x00007ffff7cbf9ff in icinga::Utility::GlobRecursive(icinga::String const&, icinga::String const&, boost::function<void (icinga::String const&)> const&, int) (path=..., pattern=..., callback=..., type=1)
    at /home/michi/coding/icinga/icinga2/lib/base/utility.cpp:609
609			callback(cpath);
(gdb) l
604
605	#endif /* _WIN32 */
606
607		std::sort(files.begin(), files.end());
608		BOOST_FOREACH(const String& cpath, files) {
609			callback(cpath);
610		}
611
612		std::sort(dirs.begin(), dirs.end());
613		BOOST_FOREACH(const String& cpath, dirs) {
(gdb) p files
$3 = std::vector of length 11, capacity 16 = {{static NPos = 18446744073709551615, m_Data = "/etc/icinga2/conf.d/agent.conf"}, {static NPos = 18446744073709551615,
    m_Data = "/etc/icinga2/conf.d/commands.conf"}, {static NPos = 18446744073709551615, m_Data = "/etc/icinga2/conf.d/downtimes.conf"}, {static NPos = 18446744073709551615,
    m_Data = "/etc/icinga2/conf.d/groups.conf"}, {static NPos = 18446744073709551615, m_Data = "/etc/icinga2/conf.d/notifications.conf"}, {static NPos = 18446744073709551615,
    m_Data = "/etc/icinga2/conf.d/satellite.conf"}, {static NPos = 18446744073709551615, m_Data = "/etc/icinga2/conf.d/services.conf"}, {static NPos = 18446744073709551615,
    m_Data = "/etc/icinga2/conf.d/templates.conf"}, {static NPos = 18446744073709551615, m_Data = "/etc/icinga2/conf.d/test.conf"}, {static NPos = 18446744073709551615,
    m_Data = "/etc/icinga2/conf.d/timeperiods.conf"}, {static NPos = 18446744073709551615, m_Data = "/etc/icinga2/conf.d/users.conf"}}
```


### Core Dump <a id="development-debug-core-dump"></a>

When the Icinga 2 daemon crashes with a `SIGSEGV` signal
a core dump file should be written. This will help
developers to analyze and fix the problem.

#### Core Dump File Size Limit <a id="development-debug-core-dump-limit"></a>

This requires setting the core dump file size to `unlimited`.


##### Systemd

```
systemctl edit icinga2.service

[Service]
...
LimitCORE=infinity

systemctl daemon-reload

systemctl restart icinga2
```

##### Init Script

```
vim /etc/init.d/icinga2
...
ulimit -c unlimited

service icinga2 restart
```

##### Verify

Verify that the Icinga 2 process core file size limit is set to `unlimited`.

```
for pid in $(pidof icinga2); do cat /proc/$pid/limits; done

...
Max core file size        unlimited            unlimited            bytes
```


#### Core Dump Kernel Format <a id="development-debug-core-dump-format"></a>

The Icinga 2 daemon runs with the SUID bit set. Therefore you need
to explicitly enable core dumps for SUID on Linux.

```bash
sysctl -w fs.suid_dumpable=2
```

Adjust the coredump kernel format and file location on Linux:

```bash
sysctl -w kernel.core_pattern=/var/lib/cores/core.%e.%p

install -m 1777 -d /var/lib/cores
```

MacOS:

```bash
sysctl -w kern.corefile=/cores/core.%P

chmod 777 /cores
```

#### Core Dump Analysis <a id="development-debug-core-dump-analysis"></a>

Once Icinga 2 crashes again a new coredump file will be written. Please
attach this file to your bug report in addition to the general details.

Simple test case for a `SIGSEGV` simulation with `sleep`:

```
ulimit -c unlimited
sleep 1800&
[1] <PID>
kill -SEGV <PID>
gdb `which sleep` /var/lib/cores/core.sleep.<PID>
(gdb) bt
rm /var/lib/cores/core.sleep.*
```

Analyzing Icinga 2:

```
gdb /usr/lib64/icinga2/sbin/icinga2 core.icinga2.<PID>
(gdb) bt
```

### LLDB as Debugger <a id="development-debug-lldb"></a>

LLDB is available on macOS with the Xcode command line tools.

```bash
xcode-select --install
```

In order to run Icinga 2 with LLDB you need to pass the binary as argument.
Since v2.11 we would attach to the umbrella process, therefore rather
attach to a running process.

```bash
# Typically the order of PIDs is: 1) umbrella 2) spawn helper 3) main process
pidof icinga2

lldb -p $(pidof icinga2 | cut -d ' ' -f3)
```

In case you'll need to attach to the main process immediately, you can delay
the forked child process and attach to the printed PID.

```
$ icinga2 daemon -DInternal.DebugWorkerDelay=120
Closed FD 6 which we inherited from our parent process.
[2020-01-29 12:22:33 +0100] information/cli: Icinga application loader (version: v2.11.0-477-gfe8701d77; debug)
[2020-01-29 12:22:33 +0100] information/RunWorker: DEBUG: Current PID: 85253. Sleeping for 120 seconds to allow lldb/gdb -p <PID> attachment.
```

```bash
lldb -p 85253
```

When lldb halts on SIGUSR2, press `c` to continue. This signal originates from the umbrella
process and can safely be ignored.


Breakpoint:

```
> b checkable.cpp:57
> b icinga::Checkable::ProcessCheckResult
```

Full backtrace:

```
> bt all
```

Select thread:

```
> thr sel 5
```

Step into:

```
> s
```

Next step:

```
> n
```

Continue:

```
> c
```

Up/down in stacktrace:

```
> up
> down
```


### Debug on Windows <a id="development-debug-windows"></a>


Whenever the application crashes, the Windows error reporting (WER) can be [configured](https://docs.microsoft.com/en-gb/windows/win32/wer/collecting-user-mode-dumps)
to create user-mode dumps.


Tail the log file with Powershell:

```
Get-Content .\icinga2.log -tail 10 -wait
```


#### Debug on Windows: Dependencies <a id="development-debug-windows-dependencies"></a>

Similar to `ldd` or `nm` on Linux/Unix.

Extract the dependent DLLs from a binary with Visual Studio's `dumpbin` tool
in Powershell:

```
C:> &'C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.22.27905\bin\Hostx64\x64\dumpbin.exe' /dependents .\debug\Bin\Debug\Debug\boosttest-test-base.exe
DEBUG:    1+  >>>> &'C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.22.27905\bin\Hostx64\x64\dumpbin.exe' /dependents .\debug\Bin\Debug\Debug\boosttest-test-base.exe
Microsoft (R) COFF/PE Dumper Version 14.22.27905.0
Copyright (C) Microsoft Corporation.  All rights reserved.


Dump of file .\debug\Bin\Debug\Debug\boosttest-test-base.exe

File Type: EXECUTABLE IMAGE

  Image has the following dependencies:

    boost_coroutine-vc142-mt-gd-x64-1_71.dll
    boost_date_time-vc142-mt-gd-x64-1_71.dll
    boost_filesystem-vc142-mt-gd-x64-1_71.dll
    boost_thread-vc142-mt-gd-x64-1_71.dll
    boost_regex-vc142-mt-gd-x64-1_71.dll
    libssl-1_1-x64.dll
    libcrypto-1_1-x64.dll
    WS2_32.dll
    dbghelp.dll
    SHLWAPI.dll
    msi.dll
    boost_unit_test_framework-vc142-mt-gd-x64-1_71.dll
    KERNEL32.dll
    SHELL32.dll
    ADVAPI32.dll
    MSVCP140D.dll
    MSWSOCK.dll
    bcrypt.dll
    VCRUNTIME140D.dll
    ucrtbased.dll

  Summary

        1000 .00cfg
       68000 .data
        B000 .idata
      148000 .pdata
      69C000 .rdata
       25000 .reloc
        1000 .rsrc
      E7A000 .text
        1000 .tls
```
