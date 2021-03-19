As we stated earlier, a module only becomes interesting with real data. What we have done wrong, however, is that our controller gets the data itself. That is not a good practice, and might cause some problems - for example if we want to use the data on the CLI.

## Our own library

We are going to create a new directory for the library we want to use in our module, following the scheme `library/<module name>`. In our case:

    mkdir -p library/Training

For our module we will use the namespace `Icinga\Modules\<module name>`. Icinga Web 2 will automatically search for any namespace created within `Training` in the newly created directory. There are some exceptions to that rule:

* `Clicommands`
* `Controllers`
* `Forms`

In a library, that helps with the task for this exercise, might be `File.php` and look like this:

```php
<?php

namespace Icinga\Module\Training;

use DirectoryIterator;

class Directory
{
    public static function listFiles($path)
    {
        $result = array();

        foreach (new DirectoryIterator($path) as $file) {
            if ($file->isDot()) continue;

            $result[] = (object) array(
                'name' => $file->getFilename(),
                'path' => $file->getPath(),
                'size' => $file->getSize(),
                'type' => $file->getType()
            );
        }
        return $result;
    }
}
```

Our controller can now retrieve the data via our small library:

```php
<?php

// ...
use Icinga\Module\Training\Directory;

class FileController extends Controller
{
    public function listAction()
    {
        $this->view->files = Directory::listFiles($this->Module()->getBaseDir());
    }
}
```

## Task

Put this, or a comparable library, in your module. Provide a view script, which displays a list of the individual files. Use `$this->escape()` in the view script, where you can't be sure the data comes from a safe source (for example, filenames).
