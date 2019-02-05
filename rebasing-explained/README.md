# Merging vs Rebasing (It's all about point of view)


## Merge Methodology 
Lets say that the `master` branch has the following commits:
```
A -> B -> C (<= master)
```

You decide to branch off `master` at commit `C` and add two more wonderful commits:
```
A -> B -> C   (<= master)
           \
            .--> A1 -> B1 (<= your local branch)
```

However, since life isn't simple, someone else manages to merge their features into master, and now your branch is missing some commits:
```
A -> B -> C -> D -> E 
           \
            .--> A1 -> B1 
```

You would like to keep your feature branch fresh and incorporate these new changes. Therefore, you `git merge master` on your feature branch. Sometimes where will be merge conflict with changes that you have on your local branch, but you resolve them and life moves on. Now you are up to date with staging, HOWEVER, you will now have a merge commit in your branch.
```
A -> B -> C -> D -> E ------.
           \                 \
            \                 v
             .--> A1 -> B1 -> C1
```

Effectively, merging will apply changes **ON TOP** of your changes (as a merge commit). Now, let's say our feature is tested and ready for master. When we merge our changes into master, our final commit history will look like:
```
A -> B -> C -> D -> E ------.---------->F
           \                 \          ^
            \                 v        /
             .--> A1 -> B1 -> C1---->./
```

Merging is a fine methodology which is easy to use, however, it makes it hard to keep your git history easy to manage. Rebase allows us to create a single timeline of events which makes it easy to figure out what features entered master.

## Rebase Methodology 

Lets see what a rebased timeline would look like. Let us assume we are branched off master and a couple commits have been added to master, so our branch is effectively behind master
```
A -> B -> C -> D -> E  (<= master)
           \
            .--> A1 -> B1 (<= your feature branch)
```

We want to incorporate changes in master, however instead of merging, we are going to rebase by typing `git rebase -i staging`. Rebase allows us to reattach our commits `A1` and `B1` to the new root of `E` (Or whatever the most recent commit in staging is). This is the same as if we branched off at commit E instead of C.
```
A -> B -> C -> D -> E
                     \                 
                      \                 
                       .--> A1 -> B1
```

This will place our changes **ON TOP** of staging. Now, when you want to incorporate your changes INTO master, you switch to the master branch with `git checkout master` and on master you type `git rebase -i feature-branch`. This will make the new head of master your feature branch.
```
A -> B -> C -> D -> E -> A1 -> B1           
```

You will notice that the branch is also gone. This is because the branch is now part of master. The git timeline is now just one long log of events. However there are some cons to this as well like going against the [golden rule](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing) as well as losing context of the actual feature branch (since its part of master).

Can we get the best of both worlds?

## Merge/Rebase Methodology (The Promised Land)

This process will be identical to the rebase methodology except for the integration into master. 
```
A -> B -> C -> D -> E  (<= master)
           \
            .--> A1 -> B1 (<= your feature branch)
```

We will `git rebase master` on our feature branch to keep it ahead of master (This will make it easy to figure out which commits are from the feature branch). 
```
A -> B -> C -> D -> E
                     \                 
                      \                 
                       .--> A1 -> B1
```

Now, we will **merge** the feature (If its more than 1 commit, squash it. Information can be found below). This will retain the history of branch **and** create a simple timeline of features into master
```
A -> B -> C -> D -> E ---------------> F
                     \                / 
                      \              /   
                       .--> A1 -> B1
```

# Conflicts
Rebase does not magically solve all of our problems. We still need to fix merge conflicts. Similar to `git merge`, you will have to resolve and add files with `git add <conflict file>`. When the conflicts are resolved, you can type `git rebase --continue`

# Quitting Rebase
You can cancel a rebase with `git rebase --abort`

# Squashing Commits

Congrats! Now you have a clean, linear flow of commits. But let's say you have a lot of commits in your feature, it would be nice to squash those commits before we merge features into staging. If each commit was a feature, there would be a clear flow of feature history.

<img src="https://blog.carbonfive.com/wp-content/uploads/2017/08/Screen-Shot-2017-07-10-at-3.39.18-PM-1-970x817.png" width="75%" height="75%">

Lets say our PR as been approved into staging. We would like to squash commits `A1` and `B1` into a single commit.
```
A -> B -> C -> D -> E
                     \                 
                      \                 
                       .--> A1 -> B1
```
```
A -> B -> C -> D -> E
                     \                 
                      \                 
                       .--> F
```

To do this, you can follow these steps.

1. Identify the last commit before the ones you want to merge. In this case, `E` is the most recent commit on staging. Run `git log` and remember the hash of `E`.
2. Run `git rebase -i <hash of E>`. This will open the default text editor with the following:
```
pick 3a823f3 A1
pick 50ac2c7 B1

# Rebase 170afb6..02e5bd1 onto 170afb6 (2 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```
3. Keep the first commit as `pick`, and change all the other pick to `squash` (or s for short):
```
pick 3a823f3 A1
squash 50ac2c7 B1

# Rebase 170afb6..02e5bd1 onto 170afb6 (2 command(s))
...
```

Congrats. You squashed the commit!. You can rename this commit with `git commit --amend`
