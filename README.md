# Git — Guide and Cheat Sheet

Table of contents
1. [Introduction & Prerequisites](#introduction--prerequisites)
2. [Quick Start](#quick-start)
3. [Basic Workflow](#basic-workflow)
4. [Inspecting History](#inspecting-history)
5. [Branching & Switching](#branching--switching)
6. [Merging & Rebasing](#merging--rebasing)
7. [Moving Commits & Cherry-pick](#moving-commits--cherry-pick)
8. [Working with Remotes (GitHub)](#working-with-remotes-github)
9. [Pull Requests & Collaboration](#pull-requests--collaboration)
10. [Stashing (Temporary Work)](#stashing-temporary-work)
11. [Undoing Mistakes & Recovery](#undoing-mistakes--recovery)
12. [Deleting Files & Branches](#deleting-files--branches)
13. [Tags & Releases](#tags--releases)
14. [.gitignore](#gitignore)
15. [Git Config & Useful Settings](#git-config--useful-settings)
16. [Git Internals (under the hood)](#git-internals-under-the-hood)
17. [Command Cheat Sheet (compact)](#command-cheat-sheet-compact)
18. [Further Reading](#further-reading)
19. [Backup / Visual Example](#backup--visual-example)

---

## Introduction & Prerequisites

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
```

---

## Quick Start

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
git clone git@github.com:owner/repo.git
# or HTTPS
git clone https://github.com/owner/repo.git
```

Set a remote and push initial branch (if needed):

```bash
git remote add origin git@github.com:owner/repo.git
git push -u origin main
```

---

## Basic Workflow

Typical flow: Working Directory → Staging Area → Local Repository → Remote

Common commands:

- Check status:

```bash
git status
```

- Stage files:

```bash
git add <file>           # stage a file
git add -A               # stage all changes (new, modified, deleted)
git add .                # stage changes in current directory
```

- Unstage a file:

```bash
git restore --staged <file>   # safer than git reset for single files
```

- Commit:

```bash
git commit -m "Short message" -m "Longer description (optional)"
git commit -am "Edit tracked files and commit in one step"  # only stages tracked files
```

- Amend last commit (fix staged content or commit message):

```bash
git add <file>
git commit --amend --no-edit   # add staged changes to previous commit without changing message
git commit --amend -m "New message"   # change message
```

Notes:
- `git commit -am` only stages changes to already tracked files.
- Use amend only for commits not yet pushed to a shared remote (or coordinate with your team if you must rewrite history).

---

## Inspecting History

Show commits:

```bash
git log
git log --oneline
git log --oneline --graph --decorate --all   # compact graphic history
git log --pretty=format:'%h %an %ad %s' --date=short
```

Show file history and blame:

```bash
git log -- <path/to/file>
git blame <path/to/file>
```

Show a commit:

```bash
git show <commit-hash>
git show --stat <commit-hash>   # files changed summary
git show -p <commit-hash>       # patch/diff
git show -p <commit-hash> -- path/to/file
```

Diffs:

```bash
git diff                # unstaged changes
git diff --staged       # staged vs HEAD
git diff HEAD~1..HEAD   # changes in last commit
```

---

## Branching & Switching

List branches:

```bash
git branch                # local branches
git branch -r             # remote branches
git branch -a             # all branches
git branch -vv            # show upstream tracking info
```

Create and switch:

```bash
git branch feature-x                # create local branch
git switch feature-x                # switch to branch (preferred)
git switch -c feature-x             # create + switch
# older:
git checkout -b feature-x
```

Switch by commit (detached HEAD):

```bash
git switch --detach <commit-hash>
```

Rename branch:

```bash
git branch -m old-name new-name    # rename local
git branch -M new-name             # rename current branch (force)
```

Show branch details (custom format):

```bash
git for-each-ref --sort=committerdate refs/heads/ --format '%(refname:short) %(committerdate:relative) - %(authorname) - %(contents:subject)'
```

---

## Merging & Rebasing

Merging:

```bash
git switch main
git merge feature-x    # merge feature into main
```

- Fast-forward merge happens if main has no new commits since branch point.
- 3-way merge creates a merge commit when both branches have diverged.

Resolving merge conflicts:
1. Use `git status` to find conflicted files.
2. Open files, look for conflict markers:

```
<<<<<<< HEAD
content in current branch
=======
content in merging branch
>>>>>>> feature-x
```

3. Edit and remove markers, `git add <file>`, then `git commit` (or `git merge --continue` if using a merge tool).

Rebasing (rewrite history to create a linear sequence):

```bash
git switch feature-x
git rebase main
# resolve conflicts if any, then
git switch main
git merge feature-x     # should be fast-forward now
git branch -d feature-x
```

Interactive rebase (squash / reorder / edit history):

```bash
git rebase -i HEAD~3
```

Warnings:
- Do not rebase commits that have been pushed to a shared/public remote unless everyone who pulled those commits coordinates and expects a history rewrite.
- If you must update pushed commits, prefer `git push --force-with-lease` over `--force`:

```bash
git push --force-with-lease origin feature-x
```

---

## Moving Commits & Cherry-pick

Cherry-pick commits onto another branch:

```bash
git switch -c bugfix-from-main
git cherry-pick <commit-hash>  # repeat for multiple commits
# or a range
git cherry-pick <start>^..<end>
```

To move commits away from a branch and remove them from the old branch:
1. Identify commits: `git log`
2. Create a new branch: `git switch -c new-branch`
3. Cherry-pick the commits.
4. Remove commits from the old branch (interactive rebase or reset — see Undoing Mistakes).

---

## Working with Remotes (GitHub)

List remotes:

```bash
git remote -v
git remote show origin
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
git push                       # push current branch to its upstream
git push -u origin feature-x   # set upstream and push
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

## Stashing (Temporary Work)

Save local changes without committing:

```bash
git stash push -m "WIP: explanation"
git stash list
git stash show stash@{0}
git stash apply stash@{0}    # keep stash entry
git stash pop                 # apply and drop stash
# stash untracked or ignored
git stash push -u -m "WIP including untracked"
git stash push -a -m "WIP including ignored"
# clear
git stash drop stash@{0}
git stash clear
```

---

## Undoing Mistakes & Recovery

Clean untracked files:

```bash
git clean -n      # dry-run
git clean -f      # remove untracked files
git clean -df     # remove untracked files & directories
```

Restore working tree files:

```bash
git restore <file>               # discard working-tree changes to file
git restore .                    # discard all working-tree changes
git restore --staged <file>      # unstage file (equivalent to git reset HEAD <file>)
```

Reset (history-rewriting, destructive operations):

- Soft reset (move HEAD, keep index & working tree):

```bash
git reset --soft <commit>
```

- Mixed reset (default; move HEAD, reset index, keep working tree):

```bash
git reset --mixed <commit>
# or simply
git reset <commit>
```

- Hard reset (move HEAD, reset index and working tree — destructive):

```bash
git reset --hard <commit>
```

Use cases and warnings:
- Avoid `git reset --hard` on branches that others use. Consider creating a backup branch before destructive resets:

```bash
git branch backup-before-reset
```

Revert (non-destructive way to undo a commit by creating a new commit that undoes a previous one):

```bash
git revert <commit-hash>
# revert multiple commits (from newest to oldest)
git revert <newer>^..<older>
```

Reflog (find lost commits):

```bash
git reflog
# recover a lost commit
git switch -c recover <commit-hash-from-reflog>
```

Cherry-pick (apply commits from another branch):

```bash
git cherry-pick <commit-hash>
```

---

## Deleting Files & Branches

Delete tracked file (stages deletion):

```bash
git rm <file>
git rm -n <file>     # dry-run
git rm -r <dir>      # recursive
git commit -m "Remove file"
```

Remove a file only from the index (keep in working tree):

```bash
git rm --cached <file>
```

Delete branches:

```bash
git branch -d <name>    # safe delete, only if merged
git branch -D <name>    # force delete
git push origin --delete <name>   # delete remote branch
```

List branches merged into current:

```bash
git branch --merged
git branch --no-merged
git branch -r --merged
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
git remote prune origin --dry-run
git remote prune origin
```

---

## Tags & Releases

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

Push tags to remote:

```bash
git push origin v1.0.0
# push all tags
git push origin --tags
```

---

## .gitignore

Create `.gitignore` in project root to ignore files/folders:

Example:

```
# Ignore specific files
file-1.txt

# Ignore folder
/bin/

# Ignore temp files
*.tmp

# Node modules
node_modules/
```

Remember: `.gitignore` itself should be committed.

If you accidentally committed files that should be ignored, remove them from the index:

```bash
git rm --cached path/to/file
git commit -m "Stop tracking file"
```

---

## Git Config & Useful Settings

Config levels:
- System: /etc/gitconfig
- Global: ~/.gitconfig
- Local: <repo>/.git/config

Show config:

```bash
git config --list
git config --global --list
cat .git/config
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

Sample ~/.gitconfig snippet:

```text
[user]
    name = YOUR NAME
    email = you@example.com
[init]
    defaultBranch = main
[commit]
    gpgsign = true
```

---

## Git Internals (under the hood)

Inspect Git objects:

```bash
git cat-file -t <object>   # type of object (commit, tree, blob, tag)
git cat-file -p <object>   # pretty print object content
git cat-file -s <object>   # size
```

Staging area / index view:

```bash
git ls-files -s
```

Show refs:

```bash
git show-ref
```

Use these when you need to debug repository problems.

---

## Command Cheat Sheet (compact)

Quick list of commonly used commands:

- Setup:
  - git config --global user.name "Name"
  - git config --global user.email "email@example.com"

- Create / clone:
  - git init
  - git clone <url>

- Work:
  - git status
  - git add <file>
  - git commit -m "msg"
  - git commit --amend
  - git restore <file>
  - git restore --staged <file>

- Branch:
  - git branch
  - git switch -c feature
  - git switch feature
  - git merge feature
  - git rebase main

- Remote:
  - git remote -v
  - git fetch --all
  - git pull --rebase
  - git push -u origin feature
  - git push --force-with-lease

- History & recovery:
  - git log --oneline --graph --decorate --all
  - git show <commit>
  - git reflog
  - git reset --hard <commit>  # destructive—use with care
  - git revert <commit>

- Stash:
  - git stash push -m "WIP"
  - git stash pop

- Tags:
  - git tag -a v1.0 -m "release"
  - git push origin v1.0

---

## Further Reading

- Pro Git book: https://git-scm.com/book/en/v2
- Git reference: https://git-scm.com/docs
- GitHub docs: https://docs.github.com

---

## Backup / Visual Example

Visual of branches / merge:

```
          (FB1) -> (FB2) -> (FB3)
           /                       \
(C1) -> (C2) -> (C3) -> (C4) -> (C5)
```
