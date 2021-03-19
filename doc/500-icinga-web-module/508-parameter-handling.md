So far we have not added any parameters to our URLs. But that's just as easy. Like on the command line, Icinga Web provides simple access to `Params`. Access is as following:

```php
<?php
$file = $this->params->get('file');
```

`shift()` and the like are available as well.

## Task

Under `training/file/show?file=<filename>` additional information about the desired file can be displayed. You can display owners, permissions, last change and mime-type - but it is also quite simple enough to display name and size in a more orderly fashion.

## Related links

In our file list we now want to link from each file to the corresponding detail area. To avoid problems with parameter escaping, we use a new helper, `qlink`:

```php
<td><?= $this->qlink(
    $file->name,
    'training/file/show',
    array('file' => $file->name)
) ?></td>
```

The first parameter is the text to be displayed, the second the link to be created, and the third is for optional parameters for this link. In the fourth parameter one can put an array with more HTML attributes.

If we now click on a file in our list, we get the corresponding details displayed. But there is also an easier way to do this. Just try putting `data-base-target="_next"` in the content-div:

    <div class="content" data-base-target="_next">

That way, we are using the multi column layout of Icinga Web for the first time - and that without much extra effort!
