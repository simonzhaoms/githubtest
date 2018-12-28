# Change history on GitHub #

* [Scenario](#scenario)
* [Solution](#solution)
* [Recap](#recap)

## Scenario ##

Suppose there are two branches:

* `master`: The main branch
* `simon`: My own branch used to test new ideas

You want to rebase `simon` onto `master`, but mistakenly **merge**d it
into `simon`, and pushed onto GitHub:

```console
$ git log --oneline --graph master simon 
* cac5b39 (HEAD -> simon, origin/simon) Move notes into separate file
* a00b4cd Explain in a different way
* e73bc58 Update README
| * 752e42c (origin/master, origin/HEAD, master) Update links
| * ee53a35 Add item for how to delete commits
| * cae99a0 Update README
|/  
* 65619f4 Initial commit

$ git merge master 
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.

$ git mergetool 
Merging:
README.md

Normal merge conflict for 'README.md':
  {local}: modified file
  {remote}: modified file

$ git status 
On branch simon
Your branch is up to date with 'origin/simon'.

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

	new file:   remove-commits.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	README.md.orig


$ rm README.md.orig 

$ git status 
On branch simon
Your branch is up to date with 'origin/simon'.

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

	new file:   remove-commits.md


$ git commit -am "Merge"
[simon aa6fcba] Merge

$ git push
Counting objects: 2, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 359 bytes | 359.00 KiB/s, done.
Total 2 (delta 0), reused 0 (delta 0)
To simonzhaomsgithub:simonzhaoms/githubtest.git
   cac5b39..aa6fcba  simon -> simon

$ git log --oneline --graph master simon 
*   aa6fcba (HEAD -> simon, origin/simon) Merge
|\  
| * 752e42c (origin/master, origin/HEAD, master) Update links
| * ee53a35 Add item for how to delete commits
| * cae99a0 Update README
* | cac5b39 Move notes into separate file
* | a00b4cd Explain in a different way
* | e73bc58 Update README
|/  
* 65619f4 Initial commit
```

## Solution ##

Now you want to revert the merge on your local repo as well as on
GitHub and do a rebase instead.  Because you are the owner of the
branch `simon` and no one else will commit onto `simon`, it is safe to
do that:

```console
$ git reset cac5b39

$ git status 
On branch simon
Your branch is behind 'origin/simon' by 4 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	remove-commits.md

nothing added to commit but untracked files present (use "git add" to track)

$ rm remove-commits.md 

$ git status 
On branch simon
Your branch is behind 'origin/simon' by 4 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean

$ ls
change-history.md  README.md

$ git push --force
Total 0 (delta 0), reused 0 (delta 0)
To simonzhaomsgithub:simonzhaoms/githubtest.git
 + aa6fcba...cac5b39 simon -> simon (forced update)

$ git status 
On branch simon
Your branch is up to date with 'origin/simon'.

nothing to commit, working tree clean

cac5b39 (HEAD -> simon, origin/simon) Move notes into separate file
a00b4cd Explain in a different way
752e42c (origin/master, origin/HEAD, master) Update links
e73bc58 Update README
ee53a35 Add item for how to delete commits
cae99a0 Update README
65619f4 Initial commit
```

## Recap ##

The key steps are:

* `git reset <commit>` to revert to the point when you want to return.
* Do what you want and commit.
* `git push --force` to change the history on your GitHub branch.
