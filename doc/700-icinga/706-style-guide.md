Overview of project files:

File Type      | File Name/Extension | Description
---------------|---------------------|-----------------------------
Header         | .hpp                | Classes, enums, typedefs inside the icinga Namespace.
Source         | .cpp                | Method implementation for class functions, static/global variables.
CMake          | CMakeLists.txt      | Build configuration, source and header file references.
CMake Source   | .cmake              | Source/Header files generated from CMake placeholders.
ITL/conf.d     | .conf               | Template library and example files as configuration
Class Compiler | .ti                 | Object classes in our own language, generates source code as `<filename>-ti.{c,h}pp`.
Lexer/Parser   | .ll, .yy            | Flex/Bison code generated into source code from CMake builds.
Docs           | .md                 | Markdown docs and READMEs.

Anything else are additional tools and scripts for developers and build systems.

All files must include the copyright header. We don't use the
current year as this implies yearly updates we don't want.

Depending on the file type, this must be a comment.

```cpp
/* Icinga 2 | (c) 2012 Icinga GmbH | GPLv2+ */
```

```bash
# Icinga 2 | (c) 2012 Icinga GmbH | GPLv2+
```

### Code Formatting <a id="development-develop-code-formatting"></a>

**Tabs instead of spaces.** Inside Visual Studio, choose to keep tabs instead of
spaces. Tabs should use 4 spaces indent by default, depending on your likings.

We follow the clang format, with some exceptions.

- Curly braces for functions and classes always start at a new line.

```cpp
String ConfigObjectUtility::EscapeName(const String& name)
{
//...
}

String ConfigObjectUtility::CreateObjectConfig(const Type::Ptr& type, const String& fullName,
	bool ignoreOnError, const Array::Ptr& templates, const Dictionary::Ptr& attrs)
{
//...
}
```

- Too long lines break at a parameter, the new line needs a tab indent.

```cpp
	static String CreateObjectConfig(const Type::Ptr& type, const String& fullName,
		bool ignoreOnError, const Array::Ptr& templates, const Dictionary::Ptr& attrs);
```

- Conditions require curly braces if it is not a single if with just one line.


```cpp
	if (s == "OK") {
		//...
	} else {
		//...
	}

	if (!n)
		return;
```

- There's a space between `if` and the opening brace `(`. Also after the closing brace `)` and opening curly brace `{`.
- Negation with `!` doesn't need an extra space.
- Else branches always start in the same line after the closing curly brace.


### Code Comments <a id="development-develop-code-comments"></a>

Add comments wherever you think that another developer will have a hard
time to understand the complex algorithm. Or you might have forgotten
it in a year and struggle again. Also use comments to highlight specific
stages in a function. Generally speaking, make things easier for the
team and external contributors.

Comments can also be used to mark additional references and TODOs.
If there is a specific GitHub issue or discussion going on,
use that information as a summary and link over to it on purpose.

- Single line comments may use `//` or `/* ... */`
- Multi line comments must use this format:

```cpp
/* Ensure to check for XY
 * This relies on the fact that ABC has been set before.
 */
```

### Function Docs <a id="development-develop-function-docs"></a>

Function header documentation must be added. The current code basis
needs rework, future functions must provide this.

Editors like CLion or Visual Studio allow you to type `/**` followed
by Enter and generate the skeleton from the implemented function.

Add a short summary in the first line about the function's purpose.
Edit the param section with short description on their intention.
The `return` value should describe the value type and additional details.

Example:

```cpp
/**
 * Reads a message from the connected peer.
 *
 * @param stream ASIO TLS Stream
 * @param yc Yield Context for ASIO
 * @param maxMessageLength maximum size of bytes read.
 *
 * @return A JSON string
 */
String JsonRpc::ReadMessage(const std::shared_ptr<AsioTlsStream>& stream, boost::asio::yield_context yc, ssize_t maxMessageLength)
```

While we can generate code docs from it, the main idea behind it is
to provide on-point docs to fully understand all parameters and the
function's purpose in the same spot.


### Header <a id="development-develop-styleguide-header"></a>

Only include other headers which are mandatory for the header definitions.
If the source file requires additional headers, add them there to avoid
include loops.

The included header order is important.

- First, include the library header `i2-<libraryname>.hpp`, e.g. `i2-base.hpp`.
- Second, include all headers from Icinga itself, e.g. `remote/apilistener.hpp`. `base` before `icinga` before `remote`, etc.
- Third, include third-party and external library headers, e.g. openssl and boost.
- Fourth, include STL headers.

### Source <a id="development-develop-styleguide-source"></a>

The included header order is important.

- First, include the header whose methods are implemented.
- Second, include all headers from Icinga itself, e.g. `remote/apilistener.hpp`. `base` before `icinga` before `remote`, etc.
- Third, include third-party and external library headers, e.g. openssl and boost.
- Fourth, include STL headers.

Always use an empty line after the header include parts.

### Namespace <a id="development-develop-styleguide-namespace"></a>

The icinga namespace is used globally, as otherwise we would need to write `icinga::Utility::FormatDateTime()`.

```cpp
using namespace icinga;
```

Other namespaces must be declared in the scope they are used. Typically
this is inside the function where `boost::asio` and variants would
complicate the code.

```cpp
	namespace ssl = boost::asio::ssl;

	auto context (std::make_shared<ssl::context>(ssl::context::sslv23));
```

### Functions <a id="development-develop-styleguide-functions"></a>

Ensure to pass values and pointers as const reference. By default, all
values will be copied into the function scope, and we want to avoid this
wherever possible.

```cpp
std::vector<EventQueue::Ptr> EventQueue::GetQueuesForType(const String& type)
```

C++ only allows to return a single value. This can be abstracted with
returning a specific class object, or with using a map/set. Array and
Dictionary objects increase the memory footprint, use them only where needed.

A common use case for Icinga value types is where a function can return
different values - an object, an array, a boolean, etc. This happens in the
inner parts of the config compiler expressions, or config validation.

The function caller is responsible to determine the correct value type
and handle possible errors.

Specific algorithms may require to populate a list, which can be passed
by reference to the function. The inner function can then append values.
Do not use a global shared resource here, unless this is locked by the caller.


### Conditions and Cases <a id="development-develop-styleguide-conditions"></a>

Prefer if-else-if-else branches. When integers are involved,
switch-case statements increase readability. Don't forget about `break` though!

Avoid using ternary operators where possible. Putting a condition
after an assignment complicates reading the source. The compiler
optimizes this anyways.

Wrong:

```cpp
	int res = s == "OK" ? 0 : s == "WARNING" ? 1;

	return res;
```

Better:

```cpp
	int res = 3;

	if (s == "OK") {
		res = 0;
	} else if (s == "WARNING") {
		res = 1;
	}
```

Even better: Create a lookup map instead of if branches. The complexity
is reduced to O(log(n)).

```cpp
	std::map<String, unsigned int> stateMap = {
		{ "OK", 1 },
		{ "WARNING", 2 }
	}

	auto it = stateMap.find(s);

	if (it == stateMap.end()) {
		return 3
	}

	return it.second;
```

The code is not as short as with a ternary operator, but one can re-use
this design pattern for other generic definitions with e.g. moving the
lookup into a utility class.

Once a unit test is written, everything works as expected in the future.

### Locks and Guards <a id="development-develop-locks-guards"></a>

Lock access to resources where multiple threads can read and write.
Icinga objects can be locked with the `ObjectLock` class.

Object locks and guards must be limited to the scope where they are needed. Otherwise we could create dead locks.

```cpp
	{
		ObjectLock olock(frame.Locals);
		for (const Dictionary::Pair& kv : frame.Locals) {
			AddSuggestion(matches, word, kv.first);
		}
	}
```

### Objects and Pointers <a id="development-develop-objects-pointers"></a>

Use shared pointers for objects. Icinga objects implement the `Ptr`
typedef returning an `intrusive_ptr` for the class object (object.hpp).
This also ensures reference counting for the object's lifetime.

Use raw pointers with care!

Some methods and classes require specific shared pointers, especially
when interacting with the Boost library.

### Value Types <a id="development-develop-styleguide-value-types"></a>

Icinga has its own value types. These provide methods to allow
generic serialization into JSON for example, and other type methods
which are made available in the DSL too.

- Always use `String` instead of `std::string`. If you need a C-string, use the `CStr()` method.
- Avoid casts and rather use the `Convert` class methods.

```cpp
	double s = static_cast<double>(v); //Wrong

	double s = Convert::ToDouble(v);   //Correct, ToDouble also provides overloads with different value types
```

- Prefer STL containers for internal non-user interfaces. Icinga value types add a small overhead which may decrease performance if e.g. the function is called 100k times.
- `Array::FromVector` and variants implement conversions, use them.

### Utilities <a id="development-develop-styleguide-utilities"></a>

Don't re-invent the wheel. The `Utility` class provides
many helper functions which allow you e.g. to format unix timestamps,
search in filesystem paths.

Also inspect the Icinga objects, they also provide helper functions
for formatting, splitting strings, joining arrays into strings, etc.

### Libraries <a id="development-develop-styleguide-libraries"></a>

2.11 depends on [Boost 1.66](https://www.boost.org/doc/libs/1_66_0/).
Use the existing libraries and header-only includes
for this specific version.

Note: Prefer C++11 features where possible, e.g. std::atomic and lambda functions.

General:

- [exception](https://www.boost.org/doc/libs/1_66_0/libs/exception/doc/boost-exception.html) (header only)
- [algorithm](https://www.boost.org/doc/libs/1_66_0/libs/algorithm/doc/html/index.html) (header only)
- [lexical_cast](https://www.boost.org/doc/libs/1_66_0/doc/html/boost_lexical_cast.html) (header only)
- [regex](https://www.boost.org/doc/libs/1_66_0/libs/regex/doc/html/index.html)
- [uuid](https://www.boost.org/doc/libs/1_66_0/libs/uuid/doc/uuid.html) (header only)
- [range](https://www.boost.org/doc/libs/1_66_0/libs/range/doc/html/index.html) (header only)
- [variant](https://www.boost.org/doc/libs/1_66_0/doc/html/variant.html) (header only)
- [multi_index](https://www.boost.org/doc/libs/1_66_0/libs/multi_index/doc/index.html) (header only)
- [function_types](https://www.boost.org/doc/libs/1_66_0/libs/function_types/doc/html/index.html) (header only)
- [circular_buffer](https://www.boost.org/doc/libs/1_66_0/doc/html/circular_buffer.html) (header only)
- [math](https://www.boost.org/doc/libs/1_66_0/libs/math/doc/html/index.html) (header only)

Events and Runtime:

- [system](https://www.boost.org/doc/libs/1_66_0/libs/system/doc/index.html)
- [thread](https://www.boost.org/doc/libs/1_66_0/doc/html/thread.html)
- [signals2](https://www.boost.org/doc/libs/1_66_0/doc/html/signals2.html) (header only)
- [program_options](https://www.boost.org/doc/libs/1_66_0/doc/html/program_options.html)
- [date_time](https://www.boost.org/doc/libs/1_66_0/doc/html/date_time.html)
- [filesystem](https://www.boost.org/doc/libs/1_66_0/libs/filesystem/doc/index.htm)

Network I/O:

- [asio](https://www.boost.org/doc/libs/1_66_0/doc/html/boost_asio.html) (header only)
- [beast](https://www.boost.org/doc/libs/1_66_0/libs/beast/doc/html/index.html) (header only)
- [coroutine](https://www.boost.org/doc/libs/1_66_0/libs/coroutine/doc/html/index.html)
- [context](https://www.boost.org/doc/libs/1_66_0/libs/context/doc/html/index.html)

Consider abstracting their usage into `*utility.{c,h}pp` files with
wrapping existing Icinga types. That also allows later changes without
rewriting large code parts.

> **Note**
>
> A new Boost library should be explained in a PR and discussed with the team.
>
> This requires package dependency changes.

If you consider an external library or code to be included with Icinga, the following
requirements must be fulfilled:

- License is compatible with GPLv2+. Boost license, MIT works, Apache is not.
- C++11 is supported, C++14 or later doesn't work
- Header only implementations are preferred, external libraries require packages on every distribution.
- No additional frameworks, Boost is the only allowed.
- The code is proven to be robust and the GitHub repository is alive, or has 1k+ stars. Good libraries also provide a user list, if e.g. Ceph is using it, this is a good candidate.


### Log <a id="development-develop-styleguide-log"></a>

Icinga allows the user to configure logging backends, e.g. syslog or file.

Any log message inside the code must use the `Log()` function.

- The first parameter is the severity level, use them with care.
- The second parameter defines the location/scope where the log
  happened. Typically we use the class name here, to better analyse
  the logs the user provide in GitHub issues and on the community
  channels.
- The third parameter takes a log message string

If the message string needs to be computed from existing values,
everything must be converted to the String type beforehand.
This conversion for every value is very expensive which is why
we try to avoid it.

Instead, use Log() with the shift operator where everything is written
on the stream and conversions are explicitly done with templates
in the background.

The trick here is that the Log object is destroyed immediately
after being constructed once. The destructor actually
evaluates the values and sends it to registers loggers.

Since flushing the stream every time a log entry occurs is
very expensive, a timer takes care of flushing the stream
every second.

> **Tip**
>
> If logging stopped, the flush timer thread may be dead.
> Inspect that with gdb/lldb.

Avoid log messages which could irritate the user. During
implementation, developers can change log levels to better
see what's going one, but remember to change this back to `debug`
or remove it entirely.


### Goto <a id="development-develop-styleguide-goto"></a>

Avoid using `goto` statements. There are rare occasions where
they are allowed:

- The code would become overly complicated within nested loops and conditions.
- Event processing and C interfaces.
- Question/Answer loops within interactive CLI commands.

### Typedef and Auto Keywords <a id="development-develop-styleguide-typedef-auto"></a>

Typedefs allow developers to use shorter names for specific types,
classes and structs.

```cpp
	typedef std::map<String, std::shared_ptr<NamespaceValue> >::iterator Iterator;
```

These typedefs should be part of the Class definition in the header,
or may be defined in the source scope where they are needed.

Avoid declaring global typedefs, unless necessary.

Using the `auto` keyword allows to ignore a specific value type.
This comes in handy with maps/sets where no specific access
is required.

The following example iterates over a map returned from `GetTypes()`.

```cpp
	for (const auto& kv : GetTypes()) {
		result.insert(kv.second);
	}
```

The long example would require us to define a map iterator, and a slightly
different algorithm.

```cpp
	typedef std::map<String, DbType::Ptr> TypeMap;
	typedef std::map<String, DbType::Ptr>::const_iterator TypeMapIterator;

	TypeMap types = GetTypes();

	for (TypeMapIterator it = types.begin(); it != types.end(); it++) {
		result.insert(it.second);
	}
```

We could also use a pair here, but requiring to know
the specific types of the map keys and values.

```cpp
	typedef std::pair<String, DbType::Ptr> kv_pair;

	for (const kv_pair& kv : GetTypes()) {
		result.insert(kv.second);
	}
```

After all, `auto` shortens the code and one does not always need to know
about the specific types. Function documentation for `GetTypes()` is
required though.



### Whitespace Cleanup <a id="development-develop-choose-editor-whitespaces"></a>

Patches must be cleaned up and follow the indent style (tabs instead of spaces).
You should also remove any trailing whitespaces.

`git diff` allows to highlight such.

```
vim $HOME/.gitconfig

[color "diff"]
        whitespace = red reverse
[core]
        whitespace=fix,-indent-with-non-tab,trailing-space,cr-at-eol
```

`vim` also can match these and visually alert you to remove them.

```
vim $HOME/.vimrc

highlight ExtraWhitespace ctermbg=red guibg=red
match ExtraWhitespace /\s\+$/
autocmd BufWinEnter * match ExtraWhitespace /\s\+$/
autocmd InsertEnter * match ExtraWhitespace /\s\+\%#\@<!$/
autocmd InsertLeave * match ExtraWhitespace /\s\+$/
autocmd BufWinLeave * call clearmatches()
```
