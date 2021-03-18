
## <a id="contributing-fork"></a> Fork the Project

[Fork the project](https://help.github.com/articles/fork-a-repo/) to your GitHub account
and clone the repository:

```bash
git clone git@github.com:dnsmichi/icinga2.git
cd icinga2
```

Add a new remote `upstream` with this repository as value.

```bash
git remote add upstream https://github.com/icinga/icinga2.git
```

You can pull updates to your fork's master branch:

```bash
git fetch --all
git pull upstream HEAD
```

Please continue to learn about [branches](CONTRIBUTING.md#contributing-branches).
