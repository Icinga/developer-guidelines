With Icinga Web 2 we want to make the integration of third party software as easy as possible. We also want to make it easy for others to use Icinga Web logic in their software.

The following call in any PHP file is enough to achieve this:

```php
<?php

require_once 'Icinga/Application/EmbeddedWeb.php';
Icinga\Application\EmbeddedWeb::start();
```

Finished! No authentication, no bootstrapping of the full web interface. But anything that exists as library code can be used.

## Task

Create an additional PHP file that embeds Icinga Web 2. Then use the directory handling library from your training module.
