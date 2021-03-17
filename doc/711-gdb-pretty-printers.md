
### GDB Pretty Printers <a id="development-advanced-gdb-pretty-printer"></a>

Install the `boost`, `python` and `icinga2` pretty printers. Absolute paths are required,
so please make sure to update the installation paths accordingly (`pwd`).

```bash
mkdir -p ~/.gdb_printers && cd ~/.gdb_printers
```

Boost Pretty Printers compatible with Python 3:

```
$ git clone https://github.com/mateidavid/Boost-Pretty-Printer.git && cd Boost-Pretty-Printer
$ git checkout python-3
$ pwd
/home/michi/.gdb_printers/Boost-Pretty-Printer
```

Python Pretty Printers:

```bash
cd ~/.gdb_printers
svn co svn://gcc.gnu.org/svn/gcc/trunk/libstdc++-v3/python
```

Icinga 2 Pretty Printers:

```bash
mkdir -p ~/.gdb_printers/icinga2 && cd ~/.gdb_printers/icinga2
wget https://raw.githubusercontent.com/Icinga/icinga2/master/tools/debug/gdb/icingadbg.py
```

Now you'll need to modify/setup your `~/.gdbinit` configuration file.
You can download the one from Icinga 2 and modify all paths.

Example on Fedora 22:

```
$ wget https://raw.githubusercontent.com/Icinga/icinga2/master/tools/debug/gdb/gdbinit -O ~/.gdbinit
$ vim ~/.gdbinit

set print pretty on

python
import sys
sys.path.insert(0, '/home/michi/.gdb_printers/icinga2')
from icingadbg import register_icinga_printers
register_icinga_printers()
end

python
import sys
sys.path.insert(0, '/home/michi/.gdb_printers/python')
from libstdcxx.v6.printers import register_libstdcxx_printers
try:
    register_libstdcxx_printers(None)
except:
    pass
end

python
import sys
sys.path.insert(0, '/home/michi/.gdb_printers/Boost-Pretty-Printer')
import boost_print
boost_print.register_printers()
end
```

If you are getting the following error when running gdb, the `libstdcxx`
printers are already preloaded in your environment and you can remove
the duplicate import in your `~/.gdbinit` file.

```
RuntimeError: pretty-printer already registered: libstdc++-v6
```

