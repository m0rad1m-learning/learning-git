# Git

## Table of contents

1. [Basic concepts](#basic-concepts)  
2. [Repository setup](#repository-setup)  
3. [Core commands](#core-commands)  
4. [Viewing commit history](#viewing-commit-history)  
5. [Branches](#branches)  
6. [Temporarily saving changes (stash)](#temporarily-saving-changes-stash)  
7. [Undoing changes](#undoing-changes)  
8. [Deleting content](#deleting-content)  
9. [Git under the hood](#git-under-the-hood)  
10. [Git ignore](#git-ignore)  
11. [Git config](#git-config)  
12. [GitHub / remotes](#github--remotes)  
13. [Pull requests](#pull-requests)  
14. [Git tags](#git-tags)  
15. [Cleaning up old branches](#cleaning-up-old-branches)  
16. [Git squash](#git-squash)  
17. [Backup (concept sketch)](#backup-concept-sketch)  

***

## Basic concepts

[ToC](#table-of-contents)

- Working tree → Staging area → (Local) Git repository → (Upstream / remote) Git repository  
- Git file lifecycle: Untracked → Modified → Staged → Unmodified  

***

## Repository setup

[ToC](#table-of-contents)

Initialize a new local repository in the current directory:

`git init`

Clone an existing remote repository:

`git clone <url>`

Clone into a specific folder:

`git clone <url> <folder>`

***

## Core commands

[ToC](#table-of-contents)

Overview of repository / status / files to commit etc.:

`git status`

Add specific changes from the working tree to the staging area:

`git add <filename>`

Add all changes from the working tree to the staging area:

`git add .`

Add only tracked files (no new untracked files):

`git add -u`

Remove specific changes from the staging area:

`git rm --cached <filename>`

Commit changes from the staging area to the local repository:

`git commit -m "<message>"`

Stage and commit tracked changes in one command:

`git commit -am "<message>"`

Change the last commit message (the last commit is replaced by a new one):

`git commit --amend -m "<message>"`

Change the content of the last commit:

```shell
git add File1.txt
git commit -m "Changes to File1.txt & File2.txt"
# File2.txt was forgotten in the staging area, so it is missing in the local repository
git add File2.txt
git commit --amend --no-edit
# File2.txt is now in the staging area. The 'amend' command moves it into the local
# repository so that the commit now includes File1.txt & File2.txt. The commit
# message itself was not changed.
```

Show differences of unstaged changes:

`git diff`

Show differences of staged changes:

`git diff --staged`

---

## Viewing commit history

[ToC](#table-of-contents)

Text-based overview:

`git log`

Text-based one-line overview of recent commits:

`git log --oneline`

One-line overview of all past commits:

`git log --pretty=oneline`

Overview with a graphical view (links between parent trees):

`git log --graph --oneline --all`

Show history of changes to a file:

`git log <file>` or `git blame <file>`

Better formatted history for a specific file:

`git log --pretty=format:'%h %an %ad %s' <file>`

### Show changes in a commit

Show which files were changed in a commit:

`git show --stat <commit-hash>`

Show all changes in a commit (diff):

`git show <commit-hash>`

Detailed view of all changes in a commit:

`git show -p <commit-hash>`

Detailed view of changes in a specific file in a commit:

`git show -p <commit-hash> -- <path-to-file>`

---

## Branches

[ToC](#table-of-contents)

### List branches

Local branches only:

`git branch` or `git branch -l`

Remote branches only:

`git branch -r`

All branches:

`git branch -a`

Show upstream / remote tracking branches of local branches:

`git branch -vv`

All local / remote tracking branches and their upstreams:

`git branch -avv`

Formatted list of local branches with detailed information:

```shell
git for-each-ref –sort=committerdate refs/heads/ 
–format ‘%(HEAD) %(align:35)%(color:blue)%(refname:short)%(color:reset)%(end) - %(color:red)%(objectname:short)%(color:reset) - %(align:40)%(contents:subject)%(end) - %(authorname) (%(color:green)%(committerdate:relative)%(color:reset))’
```

### Create / switch branches

Create a new local branch:

> Works also if the branch exists only on the remote; in that case the new local branch automatically becomes a tracking branch of the remote branch.

`git branch <branch-name>`

Create a new local branch and switch to it:

`git checkout -b <branch-name>`

(preferred in newer Git versions:)

`git switch -c <branch-name>`

Switch to an existing branch:

`git checkout <branch-name | commit-hash(first 6 chars)>`

Or (newer command):

`git switch <branch-name>`

#### Example: Checkout of a dedicated commit hash in ‘main’ branch

```text
tial state
(C1)–> (C2)–>(C3)–>(C4)–>(C5) <– HEAD, main
Checkout commit hash
git checkout c2
Final state (detached HEAD state)
(C1)–> (C2)–>(C3)–>(C4)–>(C5) <– main | HEAD
```

### Rename branches

Rename the current local branch:

`git branch -M <name>`

Rename another local branch:

`git branch -m <old-name> <new-name>`

### Merge branches (in the receiving branch, e.g. “main”)

Git merge concepts:

- **Fast-forward merge**: `main` / `HEAD` pointers are moved to the last commit of the feature branch. This only works if no commits were added to `main` after the feature branch was created.  
- **3-way merge**: A new commit is created on `main` that has two parents: the last commit on `main` and the last commit on the feature branch.

Merge the current branch with a feature branch:

`git merge <feature-branch-name>`

### Handling merge conflicts

1. Use `git status` to identify files with conflicts.  
2. Use `cat <file>` to inspect conflicts in each file. Typical conflict markers: `<<<<< HEAD`
3. Resolve the conflict in an editor of choice, e.g.  vim  or VS Code.
4. Run  git add  on resolved files and then  git commit  to confirm the merge.
   When all merge conflicts are resolved and staged, the merge continues automatically.

### Rebasing

Rebasing is an alternative to merging that can replace 3-way merges with fast-forward merges. If commits were added both to a feature branch and to main , a merge normally requires a 3-way merge. By rebasing, the starting point of the feature branch is moved to the latest commit on  main , resulting in a linear history.

```shell
git checkout feature-1
git rebase main
git checkout main
git merge feature-1
git branch -d feature-1
```

Example: Rebasing a feature branch
========
# Initial state
         (FB1)-->(FB2)-->(FB3) <-- [HEAD, feature-1]
          /
(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- [main]
 
# Commands
git checkout feature-1
git rebase main

# State
                            (FB1)-->(FB2)-->(FB3) <-- [HEAD, feature-1]
                             /
(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- [main]
