### Lexer <a id="technical-concepts-configuration-lexer"></a>

The lexer stage does not understand the DSL itself, it only
maps specific character sequences into identifiers.

This allows Icinga to detect the beginning of a string with `"`,
reading the following characters and determining the end of the
string with again `"`.

Other parts covered by the lexer a escape sequences insides a string,
e.g. `"\"abc"`.

The lexer also identifiers logical operators, e.g. `&` or `in`,
specific keywords like `object`, `import`, etc. and comment blocks.

Please check `lib/config/config_lexer.ll` for details.

Icinga uses [Flex](https://github.com/westes/flex) in the first stage.

> Flex (The Fast Lexical Analyzer)
>
> Flex is a fast lexical analyser generator. It is a tool for generating programs
> that perform pattern-matching on text. Flex is a free (but non-GNU) implementation
> of the original Unix lex program.

### Parser <a id="technical-concepts-configuration-parser"></a>

The parser stage puts the identifiers from the lexer into more
context with flow control and sequences.

The following comparison is parsed into a left term, an operator
and a right term.

```
x > 5
```

The DSL contains many elements which require a specific order,
and sometimes only a left term for example.

The parser also takes care of parsing an object declaration for
example. It already knows from the lexer that `object` marks the
beginning of an object. It then expects a type string afterwards,
and the object name - which can be either a string with double quotes
or a previously defined constant.

An opening bracket `{` in this specific context starts the object
scope, which also is stored for later scope specific variable access.

If there's an apply rule defined, this follows the same principle.
The config parser detects the scope of an apply rule and generates
Icinga 2 C++ code for the parsed string tokens.

```
assign where host.vars.sla == "24x7"
```

is parsed into an assign token identifier, and the string expression
is compiled into a new `ApplyExpression` object.

The flow control inside the parser ensures that for example `ignore where`
can only be defined when a previous `assign where` was given - or when
inside an apply for rule.

Another example are specific object types which allow assign expression,
specifically group objects. Others objects must throw a configuration error.

Please check `lib/config/config_parser.yy` for more details,
and the [language reference](17-language-reference.md#language-reference) chapter for
documented DSL keywords and sequences.

> Icinga uses [Bison](https://en.wikipedia.org/wiki/GNU_bison) as parser generator
> which reads a specification of a context-free language, warns about any parsing
> ambiguities, and generates a parser in C++ which reads sequences of tokens and
> decides whether the sequence conforms to the syntax specified by the grammar.


### Compiler <a id="technical-concepts-configuration-compiler"></a>

The config compiler initializes the scanner inside the [lexer](19-technical-concepts.md#technical-concepts-configuration-lexer)
stage.

The configuration files are parsed into memory from inside the [daemon CLI command](19-technical-concepts.md#technical-concepts-application-cli-commands-daemon)
which invokes the config validation in `ValidateConfigFiles()`. This compiles the
files into an AST expression which is executed.

At this stage, the expressions generate so-called "config items" which
are a pre-stage of the later compiled object.

`ConfigItem::CommitItems` takes care of committing the items, and doing a
rollback on failure. It also checks against matching apply rules from the previous run
and generates statistics about the objects which can be seen by the config validation.

`ConfigItem::CommitNewItems` collects the registered types and items,
and checks for a specific required order, e.g. a service object needs
a host object first.

The following stages happen then:

- **Commit**: A workqueue then commits the items in a parallel fashion for this specific type. The object gets its name, and the AST expression is executed. It is then registered into the item into `m_Object` as reference.
- **OnAllConfigLoaded**: Special signal for each object to pre-load required object attributes, resolve group membership, initialize functions and timers.
- **CreateChildObjects**: Run apply rules for this specific type.
- **CommitNewItems**: Apply rules may generate new config items, this is to ensure that they again run through the stages.

Note that the items are now committed and the configuration is validated and loaded
into memory. The final config objects are not yet activated though.

This only happens after the validation, when the application is about to be run
with `ConfigItem::ActivateItems`.

Each item has an object created in `m_Object` which is checked in a loop.
Again, the dependency order of activated objects is important here, e.g. logger features come first, then
config objects and last the checker, api, etc. features. This is done by sorting the objects
based on their type specific activation priority.

The following signals are triggered in the stages:

- **PreActivate**: Setting the `active` flag for the config object.
- **Activate**: Calls `Start()` on the object, sets the local HA authority and notifies subscribers that this object is now activated (e.g. for config updates in the DB backend).


### References <a id="technical-concepts-configuration-references"></a>

* [The Icinga Config Compiler: An Overview](https://www.netways.de/blog/2018/07/12/the-icinga-config-compiler-an-overview/)
* [A parser/lexer/compiler for the Leonardo language](https://github.com/EmilGedda/Leonardo)
* [I wrote a programming language. Here’s how you can, too.](https://medium.freecodecamp.org/the-programming-language-pipeline-91d3f449c919)
* [http://onoffswitch.net/building-a-custom-lexer/](http://onoffswitch.net/building-a-custom-lexer/)
* [Writing an Interpreter with Lex, Yacc, and Memphis](http://memphis.compilertools.net/interpreter.html)
* [Flex](https://github.com/westes/flex)
* [GNU Bison](https://www.gnu.org/software/bison/)
