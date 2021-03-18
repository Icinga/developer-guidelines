
# Your own module in the web frontend

The __'Web'__ in Icinga Web stands for its strongest suit, which we will have a look at now. And again, __convention before configuration__ applies. Built with the classic __MVC architectural pattern__, there are controllers with actions and matching view scripts for markup and display.

We have deliberately omitted the library/model separation, as each additional layer eventually increases the complexity. You could also look at the library code in many modules as a 'model', but that's a problem for the experts that come after us. Anyway, we would like to have as many well made modules as possible, ideally with a lot of reusable code, which in turn also benefits other modules.

## A first Controller

Every `action` in a `controller` automatically becomes a `route` in our web frontend. It looks something like this:

    http(s)://<host>/icingaweb/<module>/<controller>/<action>

If we want to create another 'Hello World' for our training module, we need to create the basic directory for our controllers first:

    mkdir -p training/application/controllers

Afterwards we add our controller. As you will probably have guessed already, it must be called `HelloController.php`, and be in the Controller namespace of our module:

```php
<?php

namespace Icinga\Module\Training\Controllers;

use Icinga\Web\Controller;

class HelloController extends Controller
{
    public function worldAction()
    {
    }
}
```

If we call the URL `training/application/controllers` now, we get an error message:

    Server error: script 'hello/world.phtml' not found in path
    (/usr/local/icingaweb-modules/training/application/views/scripts/)

Conveniently, it immediately tells us what we need to do next.

## Create a view script

The corresponding base directory is still missing. Since we create a view script in a dedicated file per 'action', there is one directory per 'controller':

    mkdir -p training/application/views/scripts/hello

The view script is then just like the 'action', so world.phtml:

```php
<h1>Hello World!</h1>
```

That's it, our new URL is now available. We can now use the full scope of our module and style it accordingly. And we can also use a few predefined elements. Two important classes are e.g. `controls` and `content`, for header elements and the page content.

```php
<div class="controls">
<h1>Hello World!</h1>
</div>

<div class="content">
Here you go...
</div>
```

This automatically gives even spacing to the page margins, and also makes it so that when scrolling down, the `controls` stay stationary, while the `content` scrolls. Of course, this will not be noticeable until we fill our module with more content.

## Menu entries

Menu entries in Icinga Web 2 can be personalized and/or defaulted by the administrator (*). They can also be provided by modules. What we see here is a global configuration that is located in the base directory of your own module in `configuration.php`:

```php
<?php

$this->menuSection('Training')
     ->add('Hello World')
     ->setUrl('training/hello/world');
```

### Icons for menu entries

In order to make our menu item look even better, we will add a little icon in front of it:

```php
<?php

$this->menuSection('Training')
     ->setIcon('thumbs-up')
     ->add('Hello World')
     ->setUrl('training/hello/world');
```

To have a look at the available icons, we can activate the `doc` module under `System`/`Module`. If the module is active, you can find the icon list under `Documentation`/`Developer - Style`. These icons have been embedded in a font, which has the great advantage that much fewer requests have to be made via the connection - the icons are simply 'always there'.

Alternatively, you can still use classic icons (.png etc) if you wish. This is especially useful, if you want to use a special icon (for example, a company logo) for your module, which is not included in the official Icinga Icon font:

```php
<?php

$this->menuSection('Training')->setIcon('img/icons/success.png');
```

## Adding images

If you would like to use your own images in your module, you can simply provide them under `public/img`:

    mkdir -p public/img
    wget https://icinga.com/wp-content/uploads/2016/02/icinga_icon.png
    mv icinga_icon.png public/img/

Our images are immediately accessible in the web interface, the URL pattern is as follows:

    http(s)://<icingaweb>/img/<module>/<bild>

In our case that would be: http://localhost/img/training/icinga_icon.png. This can also be used in our view scripts in the same way. Instead of creating an img tag (which of course would be possible) we use one of the many practical view helpers:

```php
...
<div class="content">
<?= $this->img('img/training/icinga_icon.png', array('title' => 'Icinga Icon')) ?> Here you go...
</div>
```

## Task

Create the URLs `training/hello/test` and `training/say/hello` and add an additional menu item. Look for a nicer icon for our training module on the internet and add it to the menu entry.

## Dashboards

Before we try to tackle more complex issues we will first provide our useful URL as a default dashboard. This can also be done in the `configuration.php`:

```php
<?php
$this->dashboard('Training')->add('Hello', 'training/hello/world');
```
