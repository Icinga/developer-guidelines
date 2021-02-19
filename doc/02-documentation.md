## <a id="contributing-patches-documentation"></a> Documentation Patches

The documentation is written in GitHub flavored [Markdown](https://guides.github.com/features/mastering-markdown/).
It is located in the `doc/` directory and can be edited with your preferred editor. You can also
edit it online on GitHub.

```bash
vim doc/2-getting-started.md
```

In order to review and test changes, you can install the [mkdocs](https://www.mkdocs.org) Python library.

```bash
pip install mkdocs
```

This allows you to start a local mkdocs viewer instance on http://localhost:8000

```bash
mkdocs serve
```

Changes on the chapter layout can be done inside the `mkdocs.yml` file in the main tree.

There also is a script to ensure that relative URLs to other sections are updated. This script
also checks for broken URLs.

```bash
./doc/update-links.py doc/*.md
```