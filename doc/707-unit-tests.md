
### Unit Tests <a id="development-develop-tests"></a>

New functions and classes must implement new unit tests. Whenever
you decide to add new functions, ensure that you don't need a complex
mock or runtime attributes in order to test them. Better isolate
code into function interfaces which can be invoked in the Boost tests
framework.

Look into the existing tests in the [test/](https://github.com/Icinga/icinga2/tree/master/test) directory
and adopt new test cases.

Specific tests require special time windows, they are only
enabled in debug builds for developers. This is the case e.g.
for testing the flapping algorithm with expected state change
detection at a specific point from now.
