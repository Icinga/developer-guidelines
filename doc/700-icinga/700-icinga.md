# Development <a id="development"></a>

This chapter provides hints on Icinga 2 debugging,
development, package builds and tests.

* [Debug Icinga 2](21-development.md#development-debug)
    * [GDB Backtrace](21-development.md#development-debug-gdb-backtrace)
    * [Core Dump](21-development.md#development-debug-core-dump)
* [Test Icinga 2](21-development.md#development-tests)
    * [Snapshot Packages (Nightly Builds)](21-development.md#development-tests-snapshot-packages)
* [Develop Icinga 2](21-development.md#development-develop)
    * [Preparations](21-development.md#development-develop-prepare)
    * [Design Patterns](21-development.md#development-develop-design-patterns)
    * [Build Tools](21-development.md#development-develop-builds-tools)
    * [Unit Tests](21-development.md#development-develop-tests)
    * [Style Guide](21-development.md#development-develop-styleguide)
* [Development Environment](21-development.md#development-environment)
    * [Linux Dev Environment](21-development.md#development-linux-dev-env)
    * [macOS Dev Environment](21-development.md#development-macos-dev-env)
    * [Windows Dev Environment](21-development.md#development-windows-dev-env)
* [Package Builds](21-development.md#development-package-builds)
    * [RPM](21-development.md#development-package-builds-rpms)
    * [DEB](21-development.md#development-package-builds-deb)
    * [Windows](21-development.md#development-package-builds-windows)
* [Continuous Integration](21-development.md#development-ci)

// GDB Pretty Printers
// replaces v
// * [Advanced Tips](21-development.md#development-advanced)

<!-- mkdocs requires 4 spaces indent for nested lists: https://github.com/Python-Markdown/markdown/issues/3 -->


## Develop Icinga 2 <a id="development-develop"></a>

Icinga 2 can be built on many platforms such as Linux, Unix and Windows.
There are limitations in terms of support, e.g. Windows is only supported for agents,
not a full-featured master or satellite.

Before you start with actual development, there is a couple of pre-requisites.

