# Commands

## Version

$ git --version
* Show version of git installed	

## Config

$ git config --global user.name "username"
$ git config --global user.email "email@email.com"
* Let's git know who you are

## Inizialize 

$ git init
* initialize git on the current working directory

## Status

$ git status
* check the status of your git repo from your current working directory
* --short flag will show a more compact version
	* Short status flags are:

		?? - Untracked files
		A - Files added to stage
		M - Modified files
		D - Deleted files

## Staging

$ git add filename.txt
* stages the file 

$ git add --all
* stages all files in the directory




## clone from a remote repo
git clone enter_repo_url_here

## Pull
Use this to update current folder with new repo changes
```
git pull origin main
```

## Log

$ git log
* View history of commits for a repository.

## Help

$ git _command_ -help (or --help) 
* options for a specific command

$ git help --all
* See all possible commands

## Link to Github
Add the repository and push. You need an access toekn as the password to log in.

```
[davidthomas@fedora PerfectDarkMode]$ git remote add origin https://github.com/tdavetech/perfectdarkmode.git
[davidthomas@fedora PerfectDarkMode]$ git branch -M main
[davidthomas@fedora PerfectDarkMode]$ git push -u origin main
```
Create a personal access Token; https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

### Make sure you are linking to HTTPS
https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories#switching-remote-urls-from-ssh-to-https

# Summary

* Version control system
* Tracking code changes
* Tracking who made changes
* Coding collaboration
* Manage projects with Repositories
* Clone a project to work on a local copy
* Staging and Committing
* Branch and Merge to allow for work on different parts and versions of a project
* Push local updates to the main project

##  How to Use

* Initialize Git on a folder, making it a Repository
* Git now creates a hidden folder to keep track of changes in that folder
* When a file is changed, added or deleted, it is considered modified
* You select the modified files you want to Stage
* The Staged files are Committed, which prompts Git to store a permanent snapshot of the files
* Git allows you to see the full history of every commit.
* You can revert back to any previous commit.
* Git does not store a separate copy of every file in every commit, but keeps track of changes made in each commit!

## Files in your Git repository folder can be in one of 2 states:

Tracked - files that Git knows about and are added to the repository
Untracked - files that are in your working directory, but not added to the repository
When you first add files to an empty repository, they are all untracked. To get Git to track them, you need to stage them, or add them to the staging environment.

## Git Staging Environment

Staged files are files that are ready to be committed to the repository you are working on.

## Git Commit

Adding commits keep track of changes. Each commit is a  change point or "save point".
You can (and should) include a message with each commit.

`$ git commit`

`$ git commit -m "add comment here"`
	* commit and add a comment
	* -m flag adds a message

`$ git commit -a -m "add message here"`
	* -a flag automatically stage every changed, already tracked file


## Commit without stage

* -a option will automatically stage every changed, already tracked file.
* Can make you commit unwanted changes

## Git Branch

* Branch = A new/separate version of the main repository.
* allow you to work on different parts of a project without impacting the main branch.
* a branch can be merged with the main project.
*  switch between branches and work on different projects without them interfering with each other.
* Low storage/overhead

`$ git branch branch-name-here`
	* create branch with name

## Checkout
 - Need to check out a branch before committing or the branch will not be included. 

`$ git checkout newImage; git commit`

create a new branch AND check it out at the same time
`$ git checkout -b [yourbranchname]`

## What is GitHub?

* Git is not the same as GitHub.
* GitHub makes tools that use Git.
* GitHub is the largest host of source code in the world, and has been owned by Microsoft since 2018.

## Git Merge
- Combine work from two different branches. 
- Creates a special commit with two parents

`$ git merge branch-name`
	- makes a new commit with two parent (Main and the chosen branch)
	- Doesn't merge the branch unless you check out the branch first.

`$ git checkout branch-name; git merge main`
	- checkout branch and merge with main

## Git Rebase

- Another way of combining work between branches
- Takes a set of commits and copies them to somewhere else
- can be used to make a nice linear sequence of commits.
- commit log / history of the repository will be a lot cleaner if only rebasing is allowed.
- moves th

`$ git rebase main`
	- adds selected branch as a commit over main
`$ git rebase branch-name`
	- You will also need to checkout main and rebase it to the last commit 

## HEAD

- the symbolic name for the currently checked out commit
- always points to the most recent commit
Detaching head
- attaching head to a commit instead of a branch

`$ git checkout commit-name`
	- checks out the commit and attaches the head


Store git credentials:
`git config credential.helper store`

Update current folder in one command `git add . && git commit -m "Message goes here" && git push`