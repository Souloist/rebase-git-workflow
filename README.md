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
