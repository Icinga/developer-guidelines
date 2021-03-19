Plugins should provide a verbose mode with `-v` or `--verbose` in order
to show more detailed log messages. This helps to debug and analyse the
flow and execution steps inside the plugin.

Ensure to add the parameter prior to implementing the check logic into
the plugin.

Example in Python taken from [check_tinkerforge](https://github.com/NETWAYS/check_tinkerforge/blob/master/check_tinkerforge.py):

```python
import argparse
import signal
import sys

if __name__ == '__main__':
    parser = argparse.ArgumentParser()

    parser.add_argument('-v', '--verbose', action='store_true')

    if args.verbose:
        print("Verbose debug output")
```
