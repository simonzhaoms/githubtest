# Nothing happend after `git rebase --continue` #

When I want to rebase current branch (`master`) onto another branch
(`dev`), I got conflits:

```console
$ git rebase origin/dev 
First, rewinding head to replay your work on top of it...
Applying: Bump version 2.0.1
Using index info to reconstruct a base tree...
M	Makefile
A	README.rst
M	mlhub/constants.py
M	setup.py
Falling back to patching base and 3-way merge...
Auto-merging setup.py
CONFLICT (content): Merge conflict in setup.py
Auto-merging mlhub/constants.py
CONFLICT (content): Merge conflict in mlhub/constants.py
CONFLICT (modify/delete): README.rst deleted in HEAD and modified in Bump version 2.0.1. Version Bump version 2.0.1 of README.rst left in tree.
Auto-merging Makefile
CONFLICT (content): Merge conflict in Makefile
error: Failed to merge in the changes.
Patch failed at 0001 Bump version 2.0.1
Use 'git am --show-current-patch' to see the failed patch

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".

$ git status 
rebase in progress; onto ee34fd1
You are currently rebasing branch 'master' on 'ee34fd1'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add/rm <file>..." as appropriate to mark resolution)

	both modified:   Makefile
	deleted by us:   README.rst
	both modified:   mlhub/constants.py
	both modified:   setup.py

no changes added to commit (use "git add" and/or "git commit -a")
```

After I resolved the conflicts via `git mergetool` or manually, I run
`git rebase --continue` to continue rebasing, then I got:

```console
$ git rebase --continue 
Applying: Bump version 2.0.1
No changes - did you forget to use 'git add'?
If there is nothing left to stage, chances are that something else
already introduced the same changes; you might want to skip this patch.

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
```

If I rerun `git rebase --continue`, I got the same:

```console
$ git rebase --continue 
Applying: Bump version 2.0.1
No changes - did you forget to use 'git add'?
If there is nothing left to stage, chances are that something else
already introduced the same changes; you might want to skip this patch.

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
```

Then what I am supposed to do is `git rebase --skip`:

```console
$ git rebase --skip 
Applying: Remove realclean.
Using index info to reconstruct a base tree...
A	docker.mk
Falling back to patching base and 3-way merge...
CONFLICT (modify/delete): docker.mk deleted in HEAD and modified in Remove realclean.. Version Remove realclean. of docker.mk left in tree.
error: Failed to merge in the changes.
Patch failed at 0002 Remove realclean.
Use 'git am --show-current-patch' to see the failed patch

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
```

What `git` is doing is to continue the rebasing from the commit where
the previous conflict occurred forward.  Then it may finish this
rebasing or encounter new conflicts like the above.  After I resolved
the new conflict, I rerun `git rebase --continue`:

```console
$ git rm docker.mk 
docker.mk: needs merge
rm 'docker.mk'

$ git status 
rebase in progress; onto ee34fd1
You are currently rebasing branch 'master' on 'ee34fd1'.
  (all conflicts fixed: run "git rebase --continue")

nothing to commit, working tree clean

$ git rebase --continue 
Applying: Remove realclean.
No changes - did you forget to use 'git add'?
If there is nothing left to stage, chances are that something else
already introduced the same changes; you might want to skip this patch.

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".

$ git status 
rebase in progress; onto ee34fd1
You are currently rebasing branch 'master' on 'ee34fd1'.
  (all conflicts fixed: run "git rebase --continue")

nothing to commit, working tree clean

$ git rebase --skip 
Applying: Note root needs to run ml configure.
Using index info to reconstruct a base tree...
M	mlhub/constants.py
Falling back to patching base and 3-way merge...
Auto-merging mlhub/constants.py
CONFLICT (content): Merge conflict in mlhub/constants.py
error: Failed to merge in the changes.
Patch failed at 0003 Note root needs to run ml configure.
Use 'git am --show-current-patch' to see the failed patch

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
```

Again on the road where `git rebase`, it encountered another new
confilt as above.  Repeating `git merge` (or resolve conficts
manually), `git rebase --continue`, `git rebase --skip`, I eventually
got:

```console
$ git rebase --continue 
Applying: Wordsmith.
```

However, this leads to divergence between my local `master` and the
remote `origin/master`:

```
$ git status 
On branch master
Your branch and 'origin/master' have diverged,
and have 83 and 6 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

nothing to commit, working tree clean
```

So `git push` will fail:

```console
$ git push
To simonzhaomsgithub:mlhubber/mlhub.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@simonzhaomsgithub:mlhubber/mlhub.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

Actually, the `dev` branch is a branch checkout from `master`, and
should be the branch where all modifications afterwards are made onto.
When a new feature is fully implemented, `master` will integrate those
modifications into itself, which is supposed to be a fast-forward
through `git rebase origin/dev` after `git checkout master`.  Why
conflicts happend instead of a fast-forward, may be we clicked the
**merge** button on GitHub or manualy executed `git merge origin/dev`
to commit a **merge**, instead of a **rebase** via **rebase** button
or `git rebase origin/dev` every time we want to incorporate the
updates from the `dev` branch.  Therefore, although the files on the
`master` and `dev` branch are the same, the history on these two
branches are different, and graphically, circles caused by merge appear
in the history of `master`.

In order to correct this and remove the circles in the history of
`master`, we can `git push --force` to completely change the history
remotely on GitHub (See [Change history of a seperate branch on
GitHub](change-history.md)):

```console
$ git push --force
Counting objects: 8, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (8/8), done.
Writing objects: 100% (8/8), 870 bytes | 870.00 KiB/s, done.
Total 8 (delta 6), reused 0 (delta 0)
remote: Resolving deltas: 100% (6/6), completed with 3 local objects.
To simonzhaomsgithub:mlhubber/mlhub.git
 + 5d25d23...c52a1c5 master -> master (forced update)
```

Then if we use `git rebase orgin/dev` instead of `git merge
origin/deve` every time we want to incorporate changes from `dev` into
`master`, then what happend must be a fast-forward.  Enjoy :smile:
:tada:

**PS** In order not to make trouble for my friends, I **should not**
use `git push --force`, so in this situation, What's better is using
`git merge origin/dev` instead of `git rebase origin/dev` and then
`git push` would succeed without resort to `git push --force`.
