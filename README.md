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

---

## Basic concepts

[ToC](#table-of-contents)

- Working tree → Staging area → (Local) Git repository → (Upstream / remote) Git repository  
- Git file lifecycle: Untracked → Modified → Staged → Unmodified  

---

## Repository setup

[ToC](#table-of-contents)

Initialize a new local repository in the current directory:

`git init`

Clone an existing remote repository:

`git clone <url>`

Clone into a specific folder:

`git clone <url> <folder>`

---

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

