# Git — Guide and Cheat Sheet

## Table of Contents

1. [Table of Contents](#table-of-contents)
2. [Introduction \& Prerequisites](#introduction--prerequisites)
3. [Quick Start](#quick-start)
4. [Basic Concepts](#basic-concepts)
5. [Repository Setup](#repository-setup)
6. [Core Commands](#core-commands)
7. [Inspecting History](#inspecting-history)
8. [Branches](#branches)
    1. [List Branches](#list-branches)
    2. [Create / Switch Branches](#create--switch-branches)
    3. [Rename Branches](#rename-branches)
    4. [Merge Branches](#merge-branches)
    5. [Resolving merge conflicts](#resolving-merge-conflicts)
    6. [Rebasing](#rebasing)
        1. [Interactive rebase / git squash](#interactive-rebase--git-squash)
9. [Moving Commits \& Cherry-pick](#moving-commits--cherry-pick)
10. [Stashing (Temporary Work)](#stashing-temporary-work)
    1. [Stash Basics](#stash-basics)
11. [Undoing Changes](#undoing-changes)
    1. [Clean untracked files](#clean-untracked-files)
    2. [Restore working tree files](#restore-working-tree-files)
    3. [Reset](#reset)
    4. [Revert](#revert)
    5. [Reflog (find lost commits)](#reflog-find-lost-commits)
12. [Deleting Files \& Branches](#deleting-files--branches)
    1. [Deleting Files](#deleting-files)
    2. [Delete Branches](#delete-branches)
13. [Git Tags](#git-tags)
14. [Git Ignore](#git-ignore)
15. [Working with Remotes (Github)](#working-with-remotes-github)
16. [Pull Requests & Collaboration](#pull-requests--collaboration)
17. [Git Config & Useful Settings](#git-config--useful-settings)
18. [Git Internals (under the hood)](#git-internals-under-the-hood)
19. [Command Cheat Sheet (compact)](#command-cheat-sheet-compact)
    1. [Setup](#setup)
    2. [Create / Clone](#create--clone)
    3. [Work](#work)
    4. [Branch](#branch)
    5. [Remote](#remote)
    6. [History \& Recovery](#history--recovery)
    7. [Stash](#stash)
    8. [Tags](#tags)
20. [Further Reading](#further-reading)
21. [Backup / Visual Example](#backup--visual-example)

---

## Introduction & Prerequisites

[back to top](#table-of-contents)

This document is a concise practical guide to common Git operations and recommended commands. It assumes you have Git installed.

Recommended global setup (one-time, per machine):

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"       # VS Code example
git config --global init.defaultBranch main
# safer push default
git config --global push.default simple
```

If you sign commits with GPG/SSH:

```bash
git config --global commit.gpgsign true
git config --global user.signingkey <your-key-id>
git config --global gpg.format ssh                 # if signing with SSH keys
```

Proxy setup (if needed):

```bash
git config --global http.proxy http://proxyuser:proxserver:port
git config --global https.proxy http://proxyuser:proxserver:port
```

---

## Quick Start

[back to top](#table-of-contents)

Create a new repository:

```bash
mkdir my-project
cd my-project
git init
# make files...
git add .
git commit -m "Initial commit"
```

Clone an existing repository:

```bash
git clone git@github.com:owner/repo.git       # when using SSH
git clone https://github.com/owner/repo.git   # when using HTTPS
```

Set a remote and push initial branch (if needed):

```bash
git remote add origin git@github.com:owner/repo.git
git push -u origin main
```

---

## Basic Concepts

[back to top](#table-of-contents)

- **Areas**: working tree → staging area → (local) git repository → (Upstream / remote) git repository
- **Git file lifecycle**: untracked / new → modified / deleted → staged → unmodified

---

## Core Commands

[back to top](#table-of-contents)

Overview of repository / status / files to commit etc.:

```bash
git status
```

Stage files:

```bash
git add <file>           # stage a file
git add -A               # stage all changes (new, modified, deleted)
git add .                # stage changes in current directory
git add -u               # stage only tracked files (no new untracked files)
```

Unstage a file:

```bash
git restore --staged <file>   # safer than git reset for single files
```

Commit:

```bash
git commit -m "Short message" -m "Longer description (optional)"
git commit -am "Edit tracked files and commit in one step"       # only stages already tracked files
```

Remove specific changes from the staging area:

```bash
git rm --cached <file>
```

Amend last commit (fix staged content or commit message):

```bash
git add <file>
git commit --amend --no-edit          # add staged changes to previous commit without changing message
git commit --amend -m "New message"   # change message
```

Notes:
- `git commit -am` only stages changes to already tracked files.
- Use amend only for commits not yet pushed to a shared remote (or coordinate with your team if you must rewrite history).

---

## Inspecting History

[back to top](#table-of-contents)

Show commit history:

```bash
git log 
git log --oneline          # text-based one-line overview of recent commits
git log --pretty=oneline   # one-line overview of all past commits
git log --pretty=format:'%h %an %ad %s' <file>  # better formatted history for
git log --graph --oneline --all --decorate   # graphical view of commit history
```

Show file history and blame:

```bash
git log <file>             # show history of changes to a file
git blame <file>           # show history of changes to a file with line-by-line annotation
```

Show a commit:

```bash
git show <commit-hash>                # show all changes in a commit (diff):
git show --stat <commit-hash>         # show which files were changed in a commit:
git show -p <commit-hash>             # detailed view of all changes in a commit:
git show -p <commit-hash> -- <file>   # detailed view of changes in a specific file 
```

Diffs:

```bash
git diff                # unstaged changes
git diff --staged       # staged vs HEAD
git diff HEAD~1..HEAD   # changes in last commit
```

---

## Branches

[back to top](#table-of-contents)

### List Branches

[back to top](#table-of-contents)

List branches:

```bash
git branch                # local branches
git branch -l             # local branches
git branch -r             # remote branches
git branch -a             # all branches
git branch -vv            # show upstream tracking info
git branch -avv           # all branches with upstream tracking info
```

Formatted list of local branches with detailed information:

```bash
# simple
git for-each-ref --sort=committerdate refs/heads/ --format '%(refname:short) %(committerdate:relative) - %(authorname) - %(contents:subject)'
```

```bash
# more advanced
git for-each-ref --sort=committerdate refs/heads/ --format '%(HEAD) %(align:35)%(color:blue)%(refname:short)%(color:reset)%(end) - %(color:red)%(objectname:short)%(color:reset) - %(align:40)%(contents:subject)%(end) - %(authorname) (%(color:green)%(committerdate:relative)%(color:reset))'
```

---

### Create / Switch Branches

[back to top](#table-of-contents)

Create and switch:

```bash
git branch feature-x                        # create local branch
git switch feature-x                        # switch to branch (preferred)
git switch -c feature-x                     # create + switch
git switch -c feature-x <start-point>       # create from specific commit
git switch -c feature-x <remote>/<branch>   # create from a remote branch, e.g. origin/main
# older:
git checkout -b feature-x
```

Switch to existing branch:

```bash
git switch <branch-name>
# older:
git checkout <branch-name>
```

Switch to existing branch by commit (detached HEAD):

```bash
git switch --detach <commit-hash>
# older:
git checkout <commit-hash>
```

```text
Example: Checkout of a Dedicated Commit Hash in 'main' Branch

# Initial state

(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- HEAD, main

# Checkout commit hash

git checkout c2

# Resulting state (detached HEAD state)

(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- main
        |
       HEAD
```

---

### Rename Branches

[back to top](#table-of-contents)

Rename branch:

```bash
git branch -m old-name new-name    # rename local
git branch -M new-name             # rename current branch (force)
```

---

### Merge Branches

[back to top](#table-of-contents)

Git merge concepts:

- **Fast-forward merge:** happens if main has no new commits since branch point.
  
  --> main / HEAD pointers are moved to the last commit of the feature branch.
- **3-way merge:** creates a merge commit when both branches have diverged

  --> a new commit is created on main with two parents: last main commit and last feature commit.

Merging:

```bash
git switch main
git merge feature-x    # merge feature into main
```

---

### Resolving merge conflicts

[back to top](#table-of-contents)

1. Use `git status` to identify files with conflicts.
2. Use `cat <file>` to inspect conflicts in each file with conflict markers.

   ```text
    <<<<<<< HEAD
    <content in current branch>
    =======
    <content in incoming merging branch>
    >>>>>>> <feature-x>
   ```

3. Resolve  conflict in editor (vim, VS Code, etc.).
4. Run `git add <file>`, then `git commit` (or `git merge --continue` if using a merge tool).

When all merge conflicts are resolved and staged, the merge continues automatically.

---

### Rebasing

[back to top](#table-of-contents)

Rebasing is an alternative to merging that can turn a divergent history into a linear history by replaying commits onto a new base.

```bash
git switch feature-1
git rebase main
# resolve conflicts if any, then
git switch main
git merge feature-1
git branch -d feature-1
```

```shell
# Example: Rebasing a Feature Branch

# Initial state:
        (FB1)-->(FB2)-->(FB3) <-- [HEAD, feature-1]
         /
(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- main

# Execute command from above

git switch feature-1
git rebase main

# State after rebase:

                             (FB1)-->(FB2)-->(FB3) <-- [HEAD, feature-1]
                              /
(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- main
```

Warnings:

- Do not rebase commits that have been pushed to a shared/public remote unless everyone who pulled those commits coordinates and expects a history rewrite.
- If you must update pushed commits, prefer `git push --force-with-lease` over `--force`:

```bash
git push --force-with-lease origin feature-x
```

#### Interactive rebase / git squash

Interactive rebase (squash / reorder / edit history):

```bash
git rebase -i HEAD~3
```

```shell
# Example: Squash multiple commits into a single commit

# Initial state:
(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- [HEAD, main]

git log --oneline

==
34a1024 (HEAD -> main) Second feature
d9d58ea Fix
f41351f Undid previous fix
0893e51 Yet another fix
231b884 Another bugfix
f08e0ce Bugfix
b9c426f First feature
==

# Execute command from above

git rebase -i HEAD~7

==
pick f08e0ce Bugfix
pick 231b884 Another bugfix
pick 0893e51 Yet another fix
pick f41351f Undid previous fix
pick d9d58ea Fix
pick 34a1024 Second feature
==

# Alter to ('s' for squash)

==
pick b9c426f First feature
s f08e0ce Bugfix
s 231b884 Another bugfix
s 0893e51 Yet another fix
s f41351f Undid previous fix
s d9d58ea Fix
pick 34a1024 Second feature
==

# In the next window, alter the commit message to

First feature

# State after rebase:

(C1)-->(C2) <-- [HEAD, main]

git log --oneline

34a1024 (HEAD -> main) Second feature
d61937b First feature
```

---

## Moving Commits & Cherry-pick

[back to top](#table-of-contents)

Cherry-pick commits onto another branch:

```bash
git log             # identify relevant commits for cherry-pick                        
git log -1         # identify relevant last commit for cherry-pick
git switch -c bugfix-from-main
git cherry-pick <commit-hash>                                     # repeat for multiple commits
git cherry-pick <commit-hash-1> <commit-hash-2> <commit-hash-3>   # multiple commits
git cherry-pick <start>^..<end>                                   # range of commits
```

To move commits away from a branch and remove them from the old branch:
1. Identify commits: `git log`
2. Create a new branch: `git switch -c new-branch`
3. Cherry-pick the commits.
4. Remove commits from the old branch (interactive rebase or reset — see Undoing Mistakes).

---

## Stashing (Temporary Work)

[back to top](#table-of-contents)

In some situations it is useful to temporarily save changes in a branch without committing them; git stash serves this purpose.


```bash
# Save changes in the working tree and staging area
git stash
git stash push -m "WIP: descriptive message / explanation"
# Stash untracked / ignored files
git stash push -u -m "WIP including untracked"
git stash push -a -m "WIP including ignored"
# Restore stashed changes
git stash pop     # apply and drop stash
git stash apply   # apply but keep stash entry
# List all stashes
git stash list
# Show contents of a stash:
git stash show
git stash show -p
# Delete stashes:
git stash drop <stash-id>
git stash clear
```

Notes:

- Multiple stashes are possible. Each stash has an ID: `stash@{#}`; `stash@{0}` is the latest.

```bash
git stash show stash@{0}
git stash apply stash@{0}
git stash drop stash@{0}
```

---

## Undoing Changes

[back to top](#table-of-contents)

---

### Clean untracked files

[back to top](#table-of-contents)

Delete files that only exist in the working tree and are untracked:

```bash
git clean -n      # dry-run
git clean -f      # delete untracked files
git clean -df     # delete untracked files and directories
```
---

### Restore working tree files

[back to top](#table-of-contents)

Remove changes to tracked files in the working tree:

```bash
git restore .         # all changes
git restore <file>    # dedicated change(s) only
git restore --staged <file>      # unstage file (equivalent to git reset HEAD <file>)
```

---

### Revert

[back to top](#table-of-contents)

Non-destructive way to undo a commit by creating a new commit that undoes a previous one:


```bash
git revert <commit-hash>
# revert multiple commits (from newest to oldest)
git revert <newer>^..<older>
```

```shell
# Example: Reverting a commit

# Initial state:

(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- [HEAD, main]

# Execute command from above

git revert c5

# State after revert:

(C1)-->(C2)-->(C3)-->(C4)-->(C5)-->(C6) <-- [HEAD, main]
```

Revert is especially useful when changes were already pushed to a remote repository, and other developers might already have pulled those.

---

### Reset

[back to top](#table-of-contents)

`git reset` moves the current branch pointer and optionally modifies staging area and working tree (destructive / *back into the past*).

> Note: Reset is always **destructive**, as it **rewrites history**. Use with caution, especially the 'hard' option. Consider creating a backup branch before destructive resets (`git branch backup-before-reset`)

**Soft** (move HEAD, keep index & working tree):

```bash
git reset --soft <HEAD | commit-hash | HEAD~5>
```

**Mixed** (default; move HEAD, reset index, keep working tree):

```bash
git reset                          # default assumes HEAD
git reset --mixed '<commit-id>'    # dedicated commit-id
```

**Hard** (move HEAD, reset index and working tree — destructive):

```bash
git reset --hard                   # default assumes HEAD
git reset --hard '<commit-id>'     # dedicated commit-id
```

```bash
# Example: Moving a commit to another branch

# Current state:
(C1)-->(C2)-->(C3)-->(C4)-->(C5) <-- [HEAD, main]

# Move last commit to new branch
git switch -c feature-branch
git reset --hard HEAD~1      # main is now at C4, feature-branch at C5  
git switch main

# Resulting state:
(C1)-->(C2)-->(C3)-->(C4) <-- [HEAD, main]
                  \
                   (C5) <-- [feature-branch]
```

---

### Reflog (find lost commits)

[back to top](#table-of-contents)

Git reflog records all reference updates. Use it to recover lost commits:

```bash
git reflog
# recover a lost commit
git switch -c recover <commit-hash-from-reflog>
```

---

## Deleting Files & Branches

[back to top](#table-of-contents)

---

### Deleting Files

[back to top](#table-of-contents)

Delete tracked content from working tree and staging area:

```bash
git rm <file>
git rm -n <file>        # dry-run
git rm -r <folder>      # recursive delete
```

The deletion becomes permanent in history only after `git commit`; before that, you can undo with `git restore .` or `git reset HEAD`.

Delete tracked content only from staging area (keep file in working tree):

```bash
git rm --cached <file>
```

---

### Delete Branches

[back to top](#table-of-contents)

Delete branches:

```bash
git branch -d <name>    # safe delete, only if merged
git branch -D <name>    # force delete
git push origin --delete <name>   # delete remote branch
```

List branches merged into current:

```bash
git branch --merged        # all merged local branches
git branch --no-merged     # all unmerged local branches
git branch -r --merged     # all merged remote branches
got branch -r --no-merged  # all unmerged remote branches
```

Cleanup merged branches (example skip main/development):

```bash
# local
git branch --merged | egrep -v "(^\*|main|development)" | xargs -r git branch -d
# remote cleanup (careful!)
git branch -r --merged | egrep -v "(^\*|origin/main|origin/development)" | sed 's/origin\///' | xargs -r -n 1 git push origin --delete
```

Prune remote-tracking branches:

```bash
git remote prune origin --dry-run  # preview
git remote prune origin            # prune stale branches
git remote update origin --prune   # prune while updating
```

---

## Git Tags

[back to top](#table-of-contents)

Git tags are references to specific commits, commonly used to mark releases like v1.0.1. Annotated tags store extra metadata and are generally preferred.

List tags:

```bash
git tag
```

Annotated tag (preferred):

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
# sign the tag (if configured)
git tag -s v1.0.0 -m "Signed release"
```

Tag a specific commit:

```bash
git tag -a v1.0.0 <commit-hash> -m "Message"
```

Force change of an existing tag:

```bash
git tag -f -a v1.0.0 -m "Updated release"
```

Push tags to remote:

```bash
git push origin v1.0.0
# push all tags
git push origin --tags
```

Force push updated tag to remote:

```bash
git push -f origin v1.0.0
```

## Git Ignore

[back to top](#table-of-contents)

The `.gitignore` file defines untracked files / directories that Git should ignore; `.gitignore` itself is tracked.

Example `.gitignore`:

```text
# Ignore dedicated file
file-1.txt

 Ignores all files with extension
*.tmp

# Ignores all files in a folder
path/to/folder/

# Ingore all files in a folder except:
path/to/folder/*
!path/to/folder/keep-this-file.txt
```

Remember: `.gitignore` itself should be committed.

If you accidentally committed files that should be ignored, remove them from the index:

```bash
git rm --cached path/to/file
git commit -m "Stop tracking file"
```

---

## Working with Remotes (GitHub)

[back to top](#table-of-contents)

List remotes:

```bash
git remote               # short list
git remote -v            # verbose
git remote show origin   # detailed info about a remote
```

Add / change remote:

```bash
git remote add origin git@github.com:owner/repo.git
git remote set-url origin git@github.com:owner/repo.git
```

Fetch (download objects and refs, no change to working tree):

```bash
git fetch
git fetch --all
git fetch origin --prune       # remove refs to deleted remote branches locally
```

Pull (fetch + merge or fetch + rebase):

```bash
git pull                       # default (merge)
git pull --rebase origin main  # fetch and rebase local commits on top of remote main
```

Push:

```bash
git push                               # push current branch to its upstream
git push -u origin feature-x           # set upstream and push
git push origin --delete branch-name   # delete remote branch
```

Prune / cleanup stale remote-tracking branches:

```bash
git remote prune origin
git remote update origin --prune
```

Safe force-push pattern:

```bash
# ensure you only overwrite what you expect
git push --force-with-lease origin feature-x
```

---

## Pull Requests & Collaboration

[back to top](#table-of-contents)

A typical collaborative flow:
1. Create a feature branch: `git switch -c feature/my-change`
2. Commit and push: `git push -u origin feature/my-change`
3. Open a pull request (PR) on GitHub from `feature/my-change` into `main` (or your target branch).
4. Request reviews, address comments, push fixes to the same branch.
5. Squash & merge or merge with commit, according to project policy.

Tips:
- Keep PRs small and focused.
- Rebase locally to clean up WIP commits before pushing (only if branch is not yet shared or team agrees).
- Use CI checks and link them in the PR.

---

## Git Config & Useful Settings

[back to top](#table-of-contents)

Git has three configuration levels:

- **System:** `/etc/gitconfig`
- **Global:** `$HOME/.gitconfig`
- **Local:** `<repo>/.git/config`

Read global config:

```bash
git config --list
git config --global --list
```

Set global user name and email:

```bash
git config --global user.name "<name>"
git config --global user.email <email>
```

Change default branch name for new repositories:

```bash
git config --global init.defaultBranch main
```

Examples of useful settings:

```bash
# safer default for pushes
git config --global push.default simple

# color and helpful abbreviations
git config --global color.ui auto
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.hist "log --oneline --graph --decorate --all"
```

Sample global `~/.gitconfig` snippet:

```text
[commit]
        gpgsign = true
[core]
        editor = vim
[gpg]
        format = ssh
[init]
        defaultbranch = main
[http]
        proxy=http://localhost:3128
[https]
        proxy=https://localhost:3128
[user]
    name = YOUR NAME
    email = you@example.com
        signingkey = ~/.ssh/id_ed25519.pub
```

---

## Git Internals (under the hood)

[back to top](#table-of-contents)

Information about Git objects:

```bash
git cat-file -p <commit-hash>   # content
git cat-file -t <commit-hash>   # type
git cat-file -s <commit-hash>   # size
```

Information about files in the staging area:

```bash
git ls-files -s
```

Show commit references and their hashes:

```bash
git show-ref                # all refs
git show-ref <branch-name>  # specific branch
```

---

## Command Cheat Sheet (compact)

[back to top](#table-of-contents)

Quick list of commonly used commands:

### Setup

```bash
git config --global user.name "Name"
git config --global user.email "email@example.com"
```

### Create / Clone

```bash
git init
git clone <url>
```

### Work

```bash
git status
git add <file>
git add -A
git commit -m "msg"
git commit --amend
git restore <file>
git restore --staged <file>
```

### Branch

```bash
git branch
git switch -c feature
git switch feature
git merge feature
git rebase main
```

### Remote

```bash
git remote -v
git fetch --all
git pull --rebase
git push -u origin feature
git push --force-with-lease
```

### History & Recovery

```bash
git log --oneline --graph --decorate --all
git show <commit>
git reflog
git reset --hard <commit>  # destructive—use with care
git revert <commit>
```

### Stash

```bash
git stash push -m "WIP"
git stash pop
```

### Tags

```bash
git tag -a v1.0 -m "release"
git push origin v1.0
```

---

## Further Reading

[back to top](#table-of-contents)

- [Pro Git book](https://git-scm.com/book/en/v2)
- [Git reference](https://git-scm.com/docs)
- [GitHub docs](https://docs.github.com)

---
