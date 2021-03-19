#### <a id="contributing-pr-review-fixes"></a> Pull Request Review Fixes

In order to amend the commit message, fix conflicts or add missing changes, you can
add your changes to the PR.

A PR is just a pointer to a different Git repository and branch.
By default, pull requests allow to push into the repository of the PR creator.

Example for [#4956](https://github.com/Icinga/icinga2/pull/4956):

At the bottom it says "Add more commits by pushing to the bugfix/persistent-comments-are-not-persistent branch on TheFlyingCorpse/icinga2."

First off, add the remote repository as additional origin and fetch its content:

```bash
git remote add theflyingcorpse https://github.com/TheFlyingCorpse/icinga2
git fetch --all
```

Checkout the mentioned remote branch into a local branch (Note: `theflyingcorpse` is the name of the remote):

```bash
git checkout theflyingcorpse/bugfix/persistent-comments-are-not-persistent -b bugfix/persistent-comments-are-not-persistent
```

Rebase, amend, squash or add your own commits on top.

Once you are satisfied, push the changes to the remote `theflyingcorpse` and its branch `bugfix/persistent-comments-are-not-persistent`.
The syntax here is `git push <remote> <localbranch>:<remotebranch>`.

```bash
git push theflyingcorpse bugfix/persistent-comments-are-not-persistent:bugfix/persistent-comments-are-not-persistent
```

In case you've changed the commit history (rebase, amend, squash), you'll need to force push. Be careful, this can't be reverted!

```bash
git push -f theflyingcorpse bugfix/persistent-comments-are-not-persistent:bugfix/persistent-comments-are-not-persistent
```

### <a id="contributing-pr-review"></a> Pull Request Review

This is meant for developers from the Icinga team who will review pull requests.
If you're not part of the team, you can of course have a look first and use it as a checklist yourself before sending it off for review!
If you want to join the development team, kindly contact us.

- Ensure that the style guide applies.
- Verify that the patch fixes a problem or linked issue, if any.
- Discuss new features with team members.
- Test the patch in your local dev environment.

If there are changes required, kindly ask for an updated patch.

Once the review is completed, merge the PR via GitHub.
