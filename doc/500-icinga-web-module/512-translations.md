For a detailed description of the translation options, we can check the documentation for the `translation` module. Here we see it step by step:

```php
<h1><?= $this->translate('My files') ?></h1>
```

    apt-get install gettext poedit
    icingacli module enable translation
    icingacli translation refresh module training de_DE
    # Translate with Poedit
    icingacli translation compile module training de_DE
