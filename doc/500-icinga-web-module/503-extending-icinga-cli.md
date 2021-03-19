The Icinga CLI was designed to provide most of the application logic in Icinga Web 2, and its modules, on the commandline. The project aims to make the creation of cronjobs, plugins, useful tools and smaller services as easy as possible.

## Own CLI Commands

Structure of the CLI commands:

    icingacli <module> <command> <action>

Creating a CLI command is very easy. The directory `application/clicommands` will create a file, whose name corresponds to the desired command:

    cd /usr/local/icingaweb-modules/training
    mkdir -p application/clicommands
    vim application/clicommands/HelloCommand.php

Here, `Hello` corresponds to the desired command, with a capital letter. The ending `Command` must ALWAYS be set.

Example command:

```php
<?php

namespace Icinga\Module\Training\Clicommands;

use Icinga\Cli\Command;

class HelloCommand extends Command
{
}
```

## Namespaces

* Namespaces help to separate modules from each other
* Each module gets a namespace, which equals the module name:

```
Icinga\Module\<Modulname>
```

* The first letter MUST be capitalized for each word
* For CLI commands, a dedicated namespace Clicommands is available

## Inheritance

All CLI commands MUST inherit the Command class in the namespace `Icinga\Cli`. This brings us a whole series of advantages, which we will discuss later. It is important that our class name corresponds to the name of the file. In our `HelloCommand.php` this would be class `HelloCommand`.

## Command Actions

Each command can provide multiple actions. Any new public method that ends with `Action` automatically becomes a CLI command action:

```php
<?php
class HelloCommand extends Command
{
    public function worldAction()
    {
        echo "Hello World!\n";
    }
}
```

## Task 1

We create a CLI Command with an action, which is executed as follows, and generates the following output:

    icingacli training say hello

## Bash Autocompletion

The Icinga CLI provides autocompletion for all modules, commands and actions. If you install Icinga Web 2 from packages, everything is already in the right place. For our test environment we will do this manually:

## Bash completion

    apt-get install bash-completion
    cp etc/bash_completion.d/icingacli /etc/bash_completion.d/
    . /etc/bash_completion

If the input is ambiguous as in `icingacli mo`, then an appropriate help text will be displayed.

## Inline Documentation for CLI Commands

Inline comments can help documenting commands and their actions. The comments' text is immediately available on the CLI, as a help text.

```php
<?php

/**
 * This is where we say hello
 *
 * The hello command allows us to be friendly to everyone
 * and their dog. That's how nice people behave!
 */
class HelloCommand extends Command
{
    /**
     * Use this to greet the world
     *
     * Greeting every single person would take some time,
     * so let's greet the whole world at once!
     */
    public function worldAction()
    {
        // ...
```

A few example combinations of how the help can be displayed:

    icingacli training
    icingacli training hello
    icingacli help training hello
    icingacli training hello world --help

The `help` command can be used before the other arguments or at any point as a parameter with `--`.

## Task 2

Create and test documentation for a `something` action for the `say` command in the `training` module!

## Command line parameters

It's possible to completely check, use and control command line parameters ourselves. Due to using inheritance, the corresponding instance of `Icinga\Cli\Params` is already available in `$this->params`. The object has a `get()` method, to which we can give the desired parameter and optionally a default value. Without default value we get `null` if the corresponding parameter is not given.

```php
<?php

// ...

    /**
     * Say hello as someone
     *
     * Usage: icingacli training hello from --from <someone>
     */
    public function fromAction()
    {
        $from = $this->params->get('from', 'Nowhere');
        echo "Hello from $from!\n";
    }
```

### Example call

    icingacli training hello from --from Nuremberg
    icingacli training hello from --from "Icinga Training"
    icingacli training hello from --help
    icingacli training hello from

## Standalone parameters

It is not necessary to assign an identifier to each parameter. If you want, you can simply chain parameters. Most conveniently, these are accessible via the `shift()` method:

```php
<?php

// ...

/**
 * Say hello as someone
 *
 * Usage: icingacli training hello from <someone>
 */
public function fromAction()
{
    $from = $this->params->shift();
    echo "Hello from $from!\n";
}
```

### Example call

    icingacli training hello from Nuremberg

## Shifting is fun

The `shift()` method behaves in the same way as you would expect from common programming languages. The first parameter of the list is returned and subsequently removed from the list. If you call `shift()` several times in succession, all existing standalone parameters are returned, until the list is empty. With `unshift()` you can undo such an action at any time.

A special case is `shift()` with an identifier (key) as a parameter. So `shift('to')` would not only return the value of the `--to` parameter, but also remove it from the params object, regardless of its position. Again, it is possible to specify a default value:

```php
<?php
// ...
$person = $this->params->shift('from', 'Nobody');
```

Of course, this also works for standalone parameters. Since we have already used the first parameter of `shift()` with the optional identifier (key), but still want to set something for the second (default value), we simply set the identifier to null here:

```php
<?php
// ...
public function fromAction()
{
    $from = $this->params->shift(null, 'Nowhere');
    echo "Hello from $from!\n";
}
```

### Example call

    icingacli training hello from Nuremberg
    icingacli training hello from
    icingacli training hello from --help

## API documentation

The Params class in the `Icinga\Cli` namespace documents other methods and their parameters. These are accessible in the API documentation for convenience. Those docs can be generated with phpDocumentor.

## Task 3

Extend the `say` command to support all of the following options:

    icingacli training say hello World
    icingacli training say hello --to World
    icingacli training say hello World --from "Icinga CLI"
    icingacli training say hello World "Icinga CLI"

## Exceptions

Icinga Web 2 wants to promote clean PHP code. This includes, among other things, that all warnings generate errors. Errors are thrown for error handling. We can just try it:

```php
<?php
// ...
use Icinga\Exception\ProgrammingError;
// ...
/**
 * This action will always fail
 */
public function brokenAction()
{
    throw new ProgrammingError('No way');
}
```

### Call

    icingacli training hello broken
    icingacli training hello broken --trace

## Exit codes

As we can see, the CLI catches all exceptions, and outputs nice and human readable error messages, along with a colored indication of the error. The exit code in this case is always 1:

    echo $?

This allows reliable evaluation of failed jobs. Only the exit code 0 stands for successful execution. Of course, everyone is free to use additional exit codes. This is done in PHP using `exit($code)`. Example:

```php
<?php
echo "CRITICAL\n";
exit(2);
```

Alternatively, Icinga Web provides the `fail()` function in the Command class. It is an abbreviation for a colored 'ERROR', a status output and `exit(1)`:

```php
<?php
$this->fail('An error occurred');
```

## Colors?

As we have just seen, the Icinga CLI can create colored output. The 'screen' class in the `Icinga\Cli` namespace provides useful help functions. We can access it in our Command classes via `$this->screen`. This way the output can be colored:

```php
<?php
echo $this->screen->colorize("Hello from $from!\n", 'lightblue');
```

As an optional third parameter, the `colorize()` function can be given a background color. For the display of the colors ANSI escape codes are used. If Icinga CLI detects that the output is NOT in a terminal/TTY, the output will not contain any colors. This ensures that e.g. when redirecting the output to a file, no disturbing special characters appear.

> To determine the terminal, PHP uses the POSIX extension. If this is not available, as a precaution the ANSI codes will not be used.

Other useful features in the Screen class are:

* `clear()` to clear the screen (used by `--watch`)
* `underline()` to underline text
* `newlines($count = 1)` to output one or more newlines
* `strlen()` to determine the character width without ANSI codes
* `center($text)` to output text centered depending on the screen width
* `getRows()` and `getColumns()` where possible, to determine the usable space
* `hasUtf8()` to query UTF8 support of the terminal

Attention: Of course, it does not work to find out that someone is traveling in a UTF8 terminal with an ISO8859 Putty.

### Task

Our `hello` action in the `say` command should output the text in color and centered both horizontally and vertically. We use `--watch` to flash the output alternately in at least two colors.
