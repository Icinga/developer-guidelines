
## <a id="contributing-testing"></a> Testing Icinga 2

Please follow the [documentation](https://icinga.com/docs/icinga2/snapshot/doc/21-development/#test-icinga-2)
for build and test instructions.

You can help test-drive the latest Icinga 2 snapshot packages inside the
[Icinga 2 Vagrant boxes](https://github.com/icinga/icinga-vagrant).


## <a id="contributing-patches-source-code"></a> Source Code Patches

Icinga 2 can be built on Linux/Unix nodes and Windows clients. In order to develop patches for Icinga 2,
you should prepare your own local build environment and know how to work with C++.

Please follow the [development documentation](https://icinga.com/docs/icinga2/latest/doc/21-development/)
for development environments, the style guide and more advanced insights.








-----------







## <a id="contributing-patches-itl-checkcommands"></a> Contribute CheckCommand Definitions

The Icinga Template Library (ITL) and its plugin check commands provide a variety of CheckCommand
object definitions which can be included on-demand.

Advantages of sending them upstream:

* Everyone can use and update/fix them.
* One single place for configuration and documentation.
* Developers may suggest updates and help with best practices.
* You don't need to care about copying the command definitions to your satellites and clients.

#### <a id="contributing-itl-checkcommands-start"></a> Where do I start?

Get to know the check plugin and its options. Read the general documentation on how to integrate
your check plugins and how to create a good CheckCommand definition.

A good command definition uses:

* Command arguments including `value`, `description`, optional: `set_if`, `required`, etc.
* Comments `/* ... */` to describe difficult parts.
* Command name as prefix for the custom attributes referenced (e.g. `disk_`)
* Default values
    * If `host.address` is involved, set a custom attribute (e.g. `ping_address`) to the default `$address$`. This allows users to override the host's address later on by setting the custom attribute inside the service apply definitions.
    * If the plugin is also capable to use ipv6, import the `ipv4-or-ipv6` template and use `$check_address$` instead of `$address$`. This allows to fall back to ipv6 if only this address is set.
    * If `set_if` is involved, ensure to specify a sane default value if required.
* Templates if there are multiple plugins with the same basic behaviour (e.g. ping4 and ping6).
* Your love and enthusiasm in making it the perfect CheckCommand.

#### <a id="contributing-itl-checkcommands-overview"></a> I have created a CheckCommand, what now?

Icinga 2 developers love documentation. This isn't just because we want to annoy anyone sending a patch,
it's a matter of making your contribution visible to the community.

Your patch should consist of 2 parts:

* The CheckCommand definition.
* The documentation bits.

[Fork the repository](https://help.github.com/articles/fork-a-repo/) and ensure that the master branch is up-to-date.

Create a new fix or feature branch and start your work.

```bash
git checkout -b feature/itl-check-printer
```

#### <a id="contributing-itl-checkcommands-add"></a> Add CheckCommand Definition to Contrib Plugins

There already exists a defined structure for contributed plugins. Navigate to `itl/plugins-contrib.d`
and verify where your command definitions fits into.

```bash
cd itl/plugins-contrib.d/
ls
```

If you want to add or modify an existing Monitoring Plugin please use `itl/command-plugins.conf` instead.

```bash
vim itl/command-plugins-conf
```

##### Existing Configuration File

Just edit it, and add your CheckCommand definition.

```bash
vim operating-system.conf
```

Proceed to the documentation.

##### New type for CheckCommand Definition

Create a new file with .conf suffix.

```bash
vim printer.conf
```

Add the file to `itl/CMakeLists.txt` in the FILES line in **alpha-numeric order**.
This ensures that the installation and packages properly include your newly created file.

```
vim CMakeLists.txt

-FILES ipmi.conf network-components.conf operating-system.conf virtualization.conf vmware.conf
+FILES ipmi.conf network-components.conf operating-system.conf printer.conf virtualization.conf vmware.conf
```

Add the newly created file to your git commit.

```bash
git add printer.conf
```

Do not commit it yet but finish with the documentation.

#### <a id="contributing-itl-checkcommands-docs"></a> Create CheckCommand Documentation

Edit the documentation file in the `doc/` directory. More details on documentation
updates can be found [here](CONTRIBUTING.md#contributing-documentation).

```bash
vim doc/10-icinga-template-library.md
```

The CheckCommand documentation should be located in the same chapter
similar to the configuration file you have just added/modified.

Create a section for your plugin, add a description and a table of parameters. Each parameter should have at least:

* optional or required
* description of its purpose
* the default value, if any

Look at the existing documentation and "copy" the same style and layout.


#### <a id="contributing-itl-checkcommands-patch"></a> Send a Patch

Commit your changes which includes a descriptive commit message.

```
git commit -av
Add printer CheckCommand definition

Explain its purpose and possible enhancements/shortcomings.

refs #existingticketnumberifany
```

Push the branch to the remote origin and create a [pull request](https://help.github.com/articles/using-pull-requests/).

```bash
git push --set-upstream origin feature/itl-check-printer
hub pull-request
```

In case developers ask for changes during review, please add them
to the branch and push those changes.
