# Rebasing, Squashing, Merging

TL;DR - use `git rebase origin/main` to get your feat/fix/chore/etc branch up to date with the mainline branch and use `git merge --squash` when merging your branch to the mainline branch.

## What does squashing commits do

On an open PR that is ready to merge in GitHub, we have the option to merge and squash commits (using `git merge --squash` under the hood). This is going to take all the commits on the branch you're working on and combine them into a single commit. This works really well when the branches that are created are always off of the mainline branch, keeping feature and bug fixes as close as possible to the mainline.

For a quick example, let's assume we have a mainline branch that has the following commits

```
* 51e67226 - feat: standardized log formatting
* 00799c5c - initial commit
```

Separately you're working on a new feature to add a health check, for example. Your branch commits look like this:

```
* 3e5bbff2 - finish adding database status check to health check endpoint
* 8d66729f - wip (end of day commit)
* e5e7fd6f - add initial simple health check
```

When you merge and squash commits, all three of those commits will be combined into a single commit with one message. You can see how useful that is on a development branch because we don't need to have commit message standards. We don't want to see "wip (end of day commit)" on the mainline version control system. This allows developers to have control over their own branch letting them commit any message they want.

So, instead of seeing this on the mainline (yes, note the merge commit): 

```
* 13671fa8 - Merge branch 'main' into feat/health-check
* 3e5bbff2 - finish adding database status check to health check endpoint
* 8d66729f - wip (end of day commit)
* e5e7fd6f - add initial simple health check
* 51e67226 - feat: standardized log formatting
* 00799c5c - initial commit
```

```
*---*---*-------* (main)
     \         /
      *---*---* (feat/health-check)
```

You get something like:

```
* b071a36f - feat: add health check endpoint
* 51e67226 - feat: standardized log formatting
* 00799c5c - initial commit
```

```
*---*---* (main)
```

_Squashing commits should not be used when you're combining multiple features and bugs into a single epic or release branch._


## What does rebasing do

Rebasing is how we keep a feature branch up to date with the mainline branch. Instead of creating a merge commit, it rewrites your commits on top of the latest mainline commits. This keeps your branch history linear without any merge commits, which matters a lot when we squash merge to main.

Let's say main has moved ahead while you've been working on your health check branch:

```
*---*---*---* (main)
     \
      *---*---* (feat/health-check)
```

Running `git rebase origin/main` rewrites your commits on top of the new main HEAD:

```
*---*---*---* (main)
             \
              *---*---* (feat/health-check)
```

Your branch looks like it was always branched off the latest main.


## What does merging do

Merging your development branch with main using `git merge origin/main` is the alternative to rebasing when keeping your branch up to date. On the surface it looks harmless, but it creates a merge commit directly on your feature branch.

Using the same example, if main moves ahead while you're working on `feat/health-check` and you run `git merge main`:

```
* 13671fa8 - Merge branch 'main' into feat/health-check
* 3e5bbff2 - finish adding database status check to health check endpoint
* 8d66729f - wip (end of day commit)
* e5e7fd6f - add initial simple health check
```

```
*---*---*---* (main)
     \       \
      *---*---* (feat/health-check)
```

That merge commit is now sitting on your feature branch. When you squash merge to main, the squash bundles everything on your branch into one commit — including that merge commit. It ends up on main even though it has nothing to do with your feature. It's just noise from syncing with upstream.

Do this enough times across enough developers and main starts looking like this:

```
* 13671fa8 - Merge branch 'main' into feat/health-check
* 3e5bbff2 - finish adding database status check to health check endpoint
* 8d66729f - wip (end of day commit)
* e5e7fd6f - add initial simple health check
* 13671fa8 - Merge branch 'main' into fix/route-missing-request-id
* e89450ba - fix: web routes missing request id
* 51e67226 - feat: standardized log formatting
* 00799c5c - initial commit
```
