# Advanced git commands

## Amending: I just committed and forgot something

Use `git commit --amend` to add stuff to the previous commit.

```
git commit -m "added feature X"
# oh no, I see a typo in file.c and fix it
git add file.c
git commit --amend
```


## Partial adding: I just want to commit some lines and not all of my changes

Use `git add -p <file>` to decide on each change whether you want to add that for the next commit.

```
# file.c has changes that I want to commit and some that I dont' want to commit (yet)
git add -p file.c
git commit -m "changes"
```


## Rebase: Move one branch on top of another

If you have a branch that you want to put on top of another branch, use `git rebase`

```
# The branches in your repo looks like this:
#  
#  master       feature/some-fix
#    |          |
#    | ---------/
#    |/
#    .
# You want the feature/some-fix branch be updated with the latest changes from master branch.

git checkout feature/some-fix
git rebase master

# Not it looks like this:
#  feature/some-fix
#    |
#    |
#  master
#    |
#    |
#    |
#    .
```

Note: When you get conflicts during rebase you must resolve them:
```
# git stopped because of conflict
# you resolve the conflict
git add some-file.c
# go on rebasing
git rebase --continue
```


## Interactive rebase: One tool to rule the git history

```
# The branches in your repo looks like this:
#  
#               feature/some-fix
#                  |
#  master       release-2.0
#    |          |
#    | ---------/
#    |/
#
# You want to rebase the commits from feature/some-fix onto the master branch
# but you don't want to include other commits from release-2.0 branch.
#
# interactive rebase starts an editor with one line per commit:
# Just drop the unwanted commits by deleting the corresponding lines.
git rebase  -i
```

During interactive rebase you can:
- Reorder commits as you like
- Change commit messages using `r` (reword) command
- Squash two or more commits into a single commit using `s` (squash) command
- Stop after any commit using `break` command which will leave you in a shell. Go on with `git rebase --continue` when you are done.


## Splitting a commit

If you just commited something and you better had split it into two or more commits:

```
# Undo the last commit
git reset HEAD^

# Now add the stuff for one of the commits
git add -p .
git commit -m "first commit"

# Repeat for other commits
git add -p .
git commit -m "second commit"
```


## Force pushing with care

Whenever you have changed the git history of your branch (e.g. using `git rebase`) you need the `--force` flag with `git push`.
It tells to basically ignore anything on the remote side and just overwrite it with your local branch.

If anybody else pushed something on the remote branch meanwhile, it gets lost.

`git push --force-with-lease` is a safer alternative because it checks whether there were any changes on the remote branch
that are not in your local repository. It will only overwrite if the remote branch has no unseen changes.


# Bisecting: Find the commit that introduced a bug

Use `git bisect` to find the commit in your commit history that introduced a bug first.

```
# start bisecting
git bisect start

# Mark one commit that is known to be good (i.e. bug not present)
git bisect good 362e1b1
# Mark one commit that is known to be bad (i.e. it has the bug)
git bisect bad 78650f9

# Git now checks out commit to test.
# Compile, run, check whether bug is there
# and mark accordingly.
git bisect good

# Git will repeat that until you have the first bad commit that introduced the bug.

# At the end clean up
git bisect reset
```


## fetch vs. pull

If you have a local branch and somebody made changes on that branch in the remote repository,
there are different ways to update your local branch:
```
#  origin/feature/foo # from remote repo
#    |
#  feature/foo
#    |          |
#    | ---------/
#    |/
#    .
```

1. fetch and merge:
Use `git fetch` to only download changes from the remote repository.
Then you can look at the origin/feature/foo manually:

```
git checkout feature/foo
git merge
```

2. pull directly
Use `git pull`  as a shortcut to fetch and merge at the same time.
```
git checkout feature/foo
git pull
```

Sometimes you have already made changes to your local branch, too:

```
#  origin/feature/foo # from remote repo
#    |
#    B       feature/foo
#    |          |
#    | ----A----/
#    |/
#    .
```

We first use `git fetch` to download the changes from the remote.

Option 1:
You want your local changes and remote changes merged in a merge commit:
```
git checkout feature/foo
git merge origin/feature/foo
# result looks like:
#
#          feature/foo
#            O
#           | |
#      ----/  \
#     /        \
#    |          |
#    B          |
#    |          |
#    | ----A----/
#    |/
#    .
```

Option 2:
You want you local changes rebased on top of the remote changes:
```
git checkout feature/foo
git rebase origin/feature/foo
# result looks like:
#
#  feature/foo
#    | 
#    A
#    |
#    B
#    |
#    |
#    |
#    .
```


Note:
- The rebase behaviour can be chosen in `git pull` shortcut, too: `git pull --rebase`.
- When working with multiple remote repositories use `git remote update` instead of `git fetch`.

