Icinga Web offers even more nice tools. One thing we still want to examine are the so-called `DataSources`. We integrate the `ArrayDatasource` and add another function to our library code:

```php
<?php

use Icinga\Data\DataArray\ArrayDatasource;

// ...

    public static function selectFiles($path)
    {
        $ds = new ArrayDatasource(self::listFiles($path));
        return $ds->select();
    }
```

Then we also make some little change our controller:

```php
<?php
$query = Directory::selectFiles(
    $this->Module()->getBaseDir()
)->order('type')->order('name');

$this->view->files = $query->fetchAll();
```

## Task 1

Rework the list so that you can sort ascending and descending via mouse click.

## Additional task

```php
<?php

$editor = Widget::create('filterEditor')->handleRequest($this->getRequest());
$query->applyFilter($editor->getFilter());
```

## Autorefresh

As a monitoring interface, it goes without saying, that Icinga Web provides a reliable and stable autorefresh functionality. This can be conveniently managed in the controllers:

```php
<?php

$this->setAutorefreshInterval(10);
```

## Task 2

Our file list should update automatically, the detail information panel should as well. Show the last modification date of a file (`$file->getMtime()`) and use the `timeSince` helper to display the time. Change a file on the hard drive and see what happens. How can that be explained?
