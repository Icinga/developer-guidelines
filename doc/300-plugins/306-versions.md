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
