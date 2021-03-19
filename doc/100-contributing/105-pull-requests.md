Once you've commited your changes, please update your local master
branch and rebase your bugfix/feature branch against it before submitting a PR.

```bash
git checkout master
git pull upstream HEAD

git checkout bugfix/notifications
git rebase master
```

Once you've resolved any conflicts, push the branch to your remote repository.
It might be necessary to force push after rebasing - use with care!

New branch:

```bash
git push --set-upstream origin bugfix/notifications
```

Existing branch:

```bash
git push -f origin bugfix/notifications
```

You can now either use the [hub](https://hub.github.com) CLI tool to create a PR, or nagivate
to your GitHub repository and create a PR there.

The pull request should again contain a telling subject and a reference
with `fixes` to an existing issue id if any. That allows developers
to automatically resolve the issues once your PR gets merged.

```
hub pull-request

<a telling subject>

fixes #1234
```

Thanks a lot for your contribution!


### <a id="contributing-rebase"></a> Rebase a Branch

If you accidentally sent in a PR which was not rebased against the upstream master,
developers might ask you to rebase your PR.

First off, fetch and pull `upstream` master.

```bash
git checkout master
git fetch --all
git pull upstream HEAD
```

Then change to your working branch and start rebasing it against master:

```bash
git checkout bugfix/notifications
git rebase master
```

If you are running into a conflict, rebase will stop and ask you to fix the problems.

```
git status

  both modified: path/to/conflict.cpp
```

Edit the file and search for `>>>`. Fix, build, test and save as needed.

Add the modified file(s) and continue rebasing.

```bash
git add path/to/conflict.cpp
git rebase --continue
```

Once succeeded ensure to push your changed history remotely.

```bash
git push -f origin bugfix/notifications
```


If you fear to break things, do the rebase in a backup branch first and later replace your current branch.

```bash
git checkout bugfix/notifications
git checkout -b bugfix/notifications-rebase

git rebase master

git branch -D bugfix/notifications
git checkout -b bugfix/notifications

git push -f origin bugfix/notifications
```

### <a id="contributing-squash"></a> Squash Commits

> **Note:**
>
> Be careful with squashing. This might lead to non-recoverable mistakes.
>
> This is for advanced Git users.

Say you want to squash the last 3 commits in your branch into a single one.

Start an interactive (`-i`)  rebase from current HEAD minus three commits (`HEAD~3`).

```bash
git rebase -i HEAD~3
```

Git opens your preferred editor. `pick` the commit in the first line, change `pick` to `squash` on the other lines.

```
pick e4bf04e47 Fix notifications
squash d7b939d99 Tests
squash b37fd5377 Doc updates
```

Save and let rebase to its job. Then force push the changes to the remote origin.

```bash
git push -f origin bugfix/notifications
```
