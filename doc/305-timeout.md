
#### Timeout <a id="service-monitoring-plugin-api-timeout"></a>

Icinga has a safety mechanism where it kills processes running for too
long. The timeout can be specified in [CheckCommand objects](09-object-types.md#objecttype-checkcommand)
or on the host/service object.

Best practice is to control the timeout in the plugin itself
and provide a clear message followed by the Unknown state.

Example in Python taken from [check_tinkerforge](https://github.com/NETWAYS/check_tinkerforge/blob/master/check_tinkerforge.py):

```python
import argparse
import signal
import sys

def handle_sigalrm(signum, frame, timeout=None):
    output('Plugin timed out after %d seconds' % timeout, 3)

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    # ... add more arguments
    parser.add_argument("-t", "--timeout", help="Timeout in seconds (default 10s)", type=int, default=10)
    args = parser.parse_args()

    signal.signal(signal.SIGALRM, partial(handle_sigalrm, timeout=args.timeout))
    signal.alarm(args.timeout)

    # ... perform the check and generate output/status
```
