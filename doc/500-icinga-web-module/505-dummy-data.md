With our web routes being all set up, we want to do something more meaningful with them. An application can look oh so dashing, but without useful content, it will quickly get boring. The usual workflow in an MVC environment looks like this: The `controller` gets its data with the aid of its `model`, and then passes it on to the `views` for displaying.

## Fill our view with data

The controller provides access to our view with `$this->view`. This way it can be populated easily:

```php
<?php

public function worldAction()
{
    $this->view->application = 'Icinga Web 2';
    $this->view->moreData = array(
        'Work'   => 'done',
        'Result' => 'fantastic'
    );
}
```

We will now expand our view script and display the submitted data:

```php
<h3>Some data...</h3>

This example is provided by <a href="http://www.icinga.com">Icinga</a> 
and based on <?= $this->application ?>.

<table>
<?php foreach ($this->moreData as $key => $val): ?>
    <tr><th><?= $key ?></th><td><?= $val ?></td></tr>
<?php endforeach ?>
</table> 
```

## Task

The contents of our module directory should be shown in form of a table under `training/list/files`.
* Note: with `$this->Module()->getBaseDir()` we can access our module directory
* More about opening directories at [http://en.php.net/opendir](http://en.php.net/opendir) 
