As a little warm up for our notebooks, we'll start off with installing Icinga Web 2:

    https://icinga.com/docs/icingaweb2/latest/doc/02-Installation/

Before we get that ball rolling, we will begin with a little introduction!

## Overview of the training

* Overview of Icinga Web 2
* Creating your own module
    * Own CLI commands
    * Working with parameters
    * Colors and other tricks
* Extending the web frontend
    * Own images
    * Own stylesheets
    * Extending the menu
    * Providing dashboards
* Working with data
    * Providing data
    * Bundle code in libraries
    * Working with parameters
    * Tips for work routines
* Configuration
* Translations
* Integration in third-party software
* Concluding remarks

## Icinga Web 2 architecture

During the development of Icinga Web 2, we built on three pillars:

* Simplicity
* Speed
* Reliability

Although we have dedicated ourselves to the DevOps movement, our target audience with Icinga Web 2 is, first and foremost, the operator - the admin. Therefore, we try to have as few dependencies as possible on external components. We forgo using some of the newest and hippest features, as it prevents things from breaking on updating to the newest versions.

The web interface is designed to be displayed on a dashboard for weeks and even months. We want to be able to rely on, that what we see, corresponds to the current state of our environment. If there are problems, they are visualized - even if they are within the application itself. When the problem is resolved, everything must continue as usual. And that without anyone having to plug in a keyboard and intervene manually.

## Libraries used

* Zend Framework 1.x
* jQuery 3
* Smaller PHP libraries
    * HTMLPurifier
    * DOMPdf
    * lessphp
    * Parsedown
* Smaller JS libraries
    * jquery-...

## Anatomy of an Icinga Web 2 module
//Todo: rename to directory structure

Icinga Web 2 follows the paradigm 'convention before configuration'. A lesson learnt from our development of Icinga Web 1: one of the best tools for XML processing is on every disk: `/bin/rm`. Those who stick to a few simple conventions will save a lot of configuration work. Basically, in Icinga Web you only have to configure paths for special cases. It is usually enough to just save a file, in the right place.

An extensive, mature module could have approximately the following structure:

    .
    └── training                Basic directory of the module
        ├── application
        │   ├── clicommands     CLI Commands
        │   ├── controllers     Web Controller
        │   ├── forms           Forms
        │   ├── locale          Translations
        │   └── views
        │       ├── helpers     View Helper
        │       └── scripts     View Scripts
        ├── configuration.php   Deploy menu, dashlets, permissions
        ├── doc                 Documentation
        ├── library
        │   └── Training        Library Code, Module Namespace
        ├── module.info         Module Metadata
        ├── public
        │   ├── css             Own CSS Code
        │   ├── img             Own Images
        │   └── js              Own JavaScript
        ├── run.php             Registration of hooks and more
        └── test
            └── php             PHP Unit Tests

We will work on our module, step by step, during this training and fill it with life.

## Source Tree preparation

To get started, we need Icinga Web 2. This can be checked out of the GIT Source Tree and used as it comes. If you then set `DocumentRoot` for a properly configured web server in the `public` directory, you can start already. For testing purposes it's even easier:

    cd /usr/local
    # If not done yet
    git clone https://git.icinga.org/icingaweb2.git
    ./icingaweb2/bin/icingacli web serve

Finished. To use the installation wizard, a token is required, for security reasons. The wizard prompts you to enter a token, which is to be generated in the CLI. This is to ensure that, between installation and setup, there is never a time when an attacker could take over an environment. For packagers this point is completely optional, the same applies to those, who roll out Icinga Web with a CM tool like Puppet: if there is a configuration on the system, you will never see the Wizard.

```
  http://localhost
```

## Manage multiple module paths

Especially those who always work with the latest version, or want to switch between GIT branches safely, usually do not want to have to change files in their working copy. Therefore, it is recommended to use several module paths in parallel from the start. This can be done in the system settings or in the configuration under `/etc/icingaweb2/config.ini`:

    [global]
    module_path= "/usr/local/icingaweb-modules:/usr/local/icingaweb2/modules"

## Installation from packages

The Icinga project builds up-to-date snapshots daily, for a wide range of operating systems, the package sources are available at [packages.icinga.org](https://packages.icinga.org/). The latest development can be checked out on [GitHub](https://github.com/Icinga/icingaweb2/).

But for our training we will use the git repository directly. And I personally also like doing that in production. Checksums for everything, changed files never go undetected, version changes happen in a fraction of a second - which package management can offer that? In addition, this procedure also demonstrates how straightforward Icinga Web 2 actually is. We did not have to change a single file in the source directory for installation. Automake, configure? What for?! The configuration is somewhere else, and WHERE it actually lies is communicated to the runtime environment.
