Those who develop a module would most likely want to be able to configure it too. The configuration for a module is stored under `/etc/icingaweb/modules/<modulename>`. Everything found in the `config.ini`, is accessible in the controller as follows:

```php
<?php
$config = $this->Config();

/*
[section]
entry = "value"
*/
echo $config->get('section', 'entry');

// Returns 'default' because 'noentry' does not exist:
echo $config->get('section', 'noentry', 'default');

// Reads from the special.ini instead of the config.ini:
$config = $this->Config('special');
```

## Task

The base path for the list controller of our training module should be configurable. If no path is configured, we continue to use our module directory.
