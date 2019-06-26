# Moving/Transfering an existing repo #

* [Scenario](#scenario)
* [Solution](#solution)


## Scenario ##

Sometimes you need to move your Git repo into a new one on GitHub.
But the new one was not created by you, and you have only write
permission to the new repo.  And you want to keep the history of the
old one, rather than copy-and-paste all files from the old repo into
the new one and make a new commit.


## Solution ##

1. Make sure you get the latest updates into your local repo from the
   old repo (it is usually called `origin` in your local repo) by `git
   pull`.

2. Add the new repo as a new remote origin (`new-origin`) into your
   local repo:

   ```console
   $ git remote add new-origin git@github.com:user/repo.git
   ```

3. Push all local branches and tags to the new repo (`new-origin`):

   ```console
   $ git push -all --force new-origin
   $ git push -tags --force new-origin
   ```

   Thus all your local branches will have a corresponding branches on
   the new repo. For example, if you have a local branch `dev` then on
   the new repo (`new-origin`), there will be a branch
   `new-origin/dev`.  And your local `master` will be push into
   `new-origin/master`.  If you mistakenly push a local branch
   (`mytest`) into the new repo (`new-repo`), you can delete it by:
   
   ```console
   $ git push new-origin --delete mytest
   ```

4. Remove the old repo (`origin`) from you local repo:

   ```console
   $ git remote rm origin  # Remove origin
   $ git remote rename new-origin origin  # Change new-origin as origin
   ```


## Reference ##

* [niksumeiko/git.migrate](https://gist.github.com/niksumeiko/8972566)
* [Transferring a repository](https://help.github.com/en/articles/transferring-a-repository)
