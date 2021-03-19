Icinga 2 heavily relies on object-oriented programming and encapsulates common
functionality into classes and objects. It also uses modern programming techniques
to e.g. work with shared pointer memory management.

Icinga 2 consists of libraries bundled into the main binary. Therefore you'll
find many code parts in the `lib/` directory wheras the actual application is
built from `icinga-app/`. Accompanied with Icinga 2, there's the Windows plugins
which are standalone and compiled from `plugins/`.

Library        | Description
---------------|------------------------------------
base           | Objects, values, types, streams, tockets, TLS, utilities, etc.
config         | Configuration compiler, expressions, etc.
cli            | CLI (sub) commands and helpers.
icinga         | Icinga specific objects and event handling.
remote         | Cluster and HTTP client/server and REST API related code.
checker        | Checker feature, check scheduler.
notification   | Notification feature, notification scheduler.
methods        | Command execution methods, plugins and built-in checks.
perfdata       | Performance data related, including Graphite, Elastic, etc.
db\_ido        | IDO database abstraction layer.
db\_ido\_mysql | IDO database driver for MySQL.
db\_ido\_pgsql | IDO database driver for PgSQL.
mysql\_shin    | Library stub for linking against the MySQL client libraries.
pgsql\_shim    | Library stub for linking against the PgSQL client libraries.

### Class Compiler <a id="development-develop-design-patterns-class-compiler"></a>

Another thing you will recognize are the `.ti` files which are compiled
by our own class compiler into actual source code. The meta language allows
developers to easily add object attributes and specify their behaviour.

Some object attributes need to be stored over restarts in the state file
and therefore have the `state` attribute set. Others are treated as `config`
attribute and automatically get configuration validation functions created.
Hidden or read-only REST API attributes are marked with `no_user_view` and
`no_user_modify`.

The most beneficial thing are getters and setters being generated. The actual object
inherits from `ObjectImpl<TYPE>` and therefore gets them "for free".

Example:

```
vim lib/perfdata/gelfwriter.ti

  [config] enable_tls;

vim lib/perfdata/gelfwriter.cpp

    if (GetEnableTls()) {
```

The logic is hidden in `tools/mkclass/` in case you want to learn more about it.
The first steps during CMake & make also tell you about code generation.
