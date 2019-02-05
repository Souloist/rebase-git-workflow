# Git Workflow
This git workflow is a combination of merging and rebasing.

## Guiding Principle (or TLDR)

	REBASE IN YOUR FEATURE BRANCH. MERGE INTO PUBLIC BRANCHES.
 
## Workflow with a single branch 

1. Branch off master and start working on your feature branch.
2. Occasionally rebase off master in order to incorporate changes into your branch.
3. When your feature is ready for QA, deploy your changes to an env.
4. If follow-ups are found in your feature, simply commit the fix in your feature branch and repear steps 3 and 4.
5. When your feature is ready to merge into master, rebase master to make sure your feature branch is ahead.
	```
	$website git(feature-branch): git checkout master
	$website git(master): git pull (IMPORTANT. Always keep master up to date.)
	$website git(feature-branch): git checkout feature-branch
	$website git(feature-branch): git rebase -i master
	```
6. When your feature is ready to merge into master, rebase **AND** squash your commits before merging into staging.
	```
	$website git(feature-branch): git checkout master
	$website git(master): git pull (IMPORTANT. Always keep master up to date.)
	$website git(feature-branch): git checkout AT-feature
	$website git(feature-branch): git rebase i master (Remember to squash!)
	```
7. Merge your feature (now a single commit) into master.
8. Congrats, you did it!

## Workflow with multiple branches

When dealing with multiple branches, it is important to define some terminology. Under the hood, Git uses merkle trees to manage commit histories.

![tree](http://shirtigo.co/wp-content/uploads/2016/03/trees.jpg)

So each branch is effectively a node in a tree. Therefore, each branch can be either considered a parent, child, or both. Parent branches are also considered **public** branches since multiple people work off that branch (feature branches).

```

A -> B -> C -> D -> E    (<= master)
           \                 
            \                 
             .--> A1 -> B1 -> C1 (<= feature A)
                         \                 
                          \
                           A2 -> A3 (<= feature B)
```

In the above scenario, feature A branched off of master, and feature B branched off of feature A. Therefore, the parent/child relationships are as follows:

* Master is the parent of feature A and is public with respect to feature A. 
* Feature A is **both** child of master but a parent to Feature B.
* Feature B is a child of Feature A.
                    
Now, we can consider a more practical situation.

Let us assume a situation where we have master, an integration branch, and multiple feature branches. The feature branches may consist of a BE and FE feature which are coupled. We would like the combine the two features in a branch which represents the full, testable feature (this being the integration branch).

```
A -> B -> C    (<= master)
           \                 
            \                 
             . (<= Integration branch)
             |\                 
             |  \
             |   A2 -> B2 (<= feature 1)
             | 
             A3 -> B3 (<= feature 2)
```

At the beginning, all these branches have branched off master and the integration branch is effectively the same as master. Master and the integration branch are considered public branches. As time progresses, one of these features may finish before the other and will be incorporated into the integration branch. Because both feature branches are children of the integration branch, these children must be **merged** into the parent.

```
A -> B -> C  (<= master)
           \                 
            \                 
             .-------------> D  (<= Integration branch)
             |\             /     
             |  \         /
             |   A2 --> B2  (<= feature 1)
             | 
             A3 -> B3 (<= feature 2)
```

Since feature 1 was merged into the integration branch, the integration branch is ahead of where feature 2 originally branched off. Therefore, feature 2 needs to rebase off of the integration branch since feature 2 is a child of the integration branch

```
A -> B -> C  (<= master)
           \                 
            \                 
             .-------------> D  (<= Integration branch)
              \             / \    
                \         /     \   
  (feature 1 =>) A2 --> B2       A3 -> B3 (<= feature 2)           
```

Let's say master gets ahead of the integration branch since features are constantly being released. How do we keep the integration branch up to date? 

```
A -> B -> C - E  (<= master)
           \                 
            \                 
             .-------------> D  (<= Integration branch)
              \             / \    
                \         /     \   
  (feature 1 =>) A2 --> B2       A3 -> B3 (<= feature 2)          
```

Since the integration branch is a child with respect to master, it must rebase to the new head of master.
```
A -> B -> C - E  (<= master)
               \                 
                \                 
                 .-------------> D'  (<= Integration branch)
                  
                  X            X  X   
                   \         /     \   
  (feature 1 =>)    A2 --> B2       A3 -> B3 (<= feature 2)          
```

However, you will notice that since rebasing creates a new commit, the children branches will now have a detached head. In order to fix the history, all children branches much rebase off of their parents. 

```
A -> B -> C - E  (<= master)
               \                 
                \                 
                 .--------> D'  (<= Integration branch)
                            | \   
                            |  \   
                            |   A3' -> B3' (<= feature 2)   
                            . (<= feature 1) 
```

Notice how feature 2 is now up to date with both master and the integration branch. However, since feature 1 was already merged into the integration branch, when the rebase occurs, there are no more commits in feature 1 since they are already in the integration branch. 

**IMPORTANT**
This is a key distinction with rebasing. Once a child branch gets merged into its parent, the rebase will effectively destroy the branch. Therefore, merging into an integration branch is the end of the life cycle for a branch. Make sure to merge changes of the feature branch (for testing sake) into qa **BEFORE** it gets into the integration branch. If you want to add new changes, create a new branch off the integration branch.

Let's finish this up by merging feature 2 into the integration branch.
```
A -> B -> C - E  (<= master)
               \                 
                \                 
                 .--------> D' ------------ F (<= Integration branch)
                              \            /
                               \          /
                                A3' -> B3' (<= feature 2)   
                       
```

Now, assuming the integration branch is ready to be deployed, let's finally squash the integration branch into a single commit.
```
A -> B -> C - E  (<= master)
               \                 
                \                 
                 .--------> DF' (<= Integration branch)                       
```

And we can finally merge the full feature into master.
```
A -> B -> C - E  ------> G  (<= master)
               \        /         
                \      /           
                 .-> DF' (<= Integration branch)                       
```

## Common Pitfalls

Let us revisit the situation where master is ahead of the integration branch. What if instead of rebasing the integration branch off of master, i decide to rebase one of the feature branches? After all, i want those changes in the my branch!

```
A -> B -> C - E  (<= master)
           \                 
            \                 
             .-------------> D  (<= Integration branch)
              \             / \    
                \         /     \   
  (feature 1 =>) A2 --> B2       A3 -> B3 (<= feature 2)          
```

This is a common misconception and could cause disastrous results! ALWAYS rebase your parent, never your grandparent or any node further up the hierarchy. (unless you know what you are doing). 
```
A -> B -> C --------------> E  (<= master)
           \                 \ 
            \                 \ 
             .-----------> D   \
              \           /     \    
                \        /       \   
  (feature 1 =>) A2 -> B2        A3' -> B3' (<= feature 2)          
```

Remember rebasing is simply just taking the commits in your feature and planting them off a new head. In this case, rebasing master will readjust the head to master! This will effectively wipe out any changes in commit D since you are no longer branched off the integration branch. 
