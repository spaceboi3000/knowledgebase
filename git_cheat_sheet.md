# Git Cheat Sheet

## Setup
- **Set username**: `git config --global user.name "Your Name"`
- **Set email**: `git config --global user.email "you@example.com"`
- **Check config**: `git config --list`

## Create / Clone
- **Initialize repository**: `git init`
- **Clone repository**: `git clone <url>`

## Basic Workflow
- **Check status**: `git status`
- **Add files**: `git add <file>` or `git add .`
- **Commit changes**: `git commit -m "message"`
- **View log**: `git log` / `git log --oneline`

## Branching
- **List branches**: `git branch`
- **Create branch**: `git branch <name>`
- **Switch branch**: `git switch <name>` or `git checkout <name>`
- **Create + switch**: `git switch -c <name>`
- **Delete branch**: `git branch -d <name>`

## Merging & Rebase
- **Merge branch**: `git merge <name>`
- **Rebase**: `git rebase <branch>`
- **Abort rebase**: `git rebase --abort`

## Remote Repositories
- **Add remote**: `git remote add origin <url>`
- **Show remotes**: `git remote -v`
- **Push branch**: `git push origin <branch>`
- **Pull changes**: `git pull`
- **Fetch**: `git fetch`

## Undo / Fix
- **Undo staged file**: `git restore --staged <file>`
- **Undo modifications**: `git restore <file>`
- **Reset to commit**:
  - Soft: `git reset --soft <commit>`
  - Mixed: `git reset --mixed <commit>`
  - Hard: `git reset --hard <commit>`

## Stash
- **Stash changes**: `git stash`
- **List stashes**: `git stash list`
- **Apply stash**: `git stash apply`
- **Drop stash**: `git stash drop`

## Tags
- **Create tag**: `git tag <name>`
- **Push tags**: `git push --tags`
- **List tags**: `git tag`

## Useful Shortcuts
- **Show changes**: `git diff`
- **Show staged changes**: `git diff --cached`
- **Show file history**: `git log -- <file>`
- **One-line graph**: `git log --oneline --graph --decorate --all`

## Clean
- **Remove untracked files**: `git clean -f`
- **Preview clean**: `git clean -n`


## Default Settings
- **Default editor**: `git config --global core.editor "nano"`
- **Default branch name**: `git config --global init.defaultBranch main`
- **Default merge tool**: `git config --global merge.tool vimdiff`
- **Default diff tool**: `git config --global diff.tool vimdiff`
- **Enable colored output**: `git config --global color.ui auto`

---

# Git Cheat Sheet: Multiple Accounts Setup

## Overview
Configure your PC to use multiple Git identities (e.g., work + personal) using SSH keys and per-directory config.

## Step 1: Create SSH Keys
```
ssh-keygen -t ed25519 -C "work@example.com" -f ~/.ssh/id_ed25519_work
ssh-keygen -t ed25519 -C "personal@example.com" -f ~/.ssh/id_ed25519_personal
```

## Step 2: Add Keys to SSH Agent
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_work
ssh-add ~/.ssh/id_ed25519_personal
```

## Step 3: Configure ~/.ssh/config
```
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work

Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
```

## Step 4: Use Different Remotes
- **Work repo**:
```
git clone git@github-work:org/work-repo.git
```
- **Personal repo**:
```
git clone git@github-personal:me/personal-repo.git
```

## Step 5: Set Local Identity Per Repo
```
git config user.name "Work Name"
git config user.email "work@example.com"
```
Or for personal:
```
git config user.name "Personal Name"
git config user.email "personal@example.com"
```

## Step 6: Optional — Global Fallback Identity
```
git config --global user.name "Default Name"
git config --global user.email "default@example.com"
```

---


# Advanced Git Cheat Sheet

## Inspect & Debug
- **Reflog (full history of HEAD movements)**: `git reflog`
- **Show commit details**: `git show <commit>`
- **Blame file**: `git blame <file>`
- **Bisect (find buggy commit)**:
```
git bisect start
git bisect bad
git bisect good <commit>
# test each step
git bisect reset
```

## Patch & Apply
- **Create patch from commit**: `git format-patch -1 <commit>`
- **Apply patch**: `git apply <patchfile>`

## Cherry-Picking
- **Apply specific commit to current branch**: `git cherry-pick <commit>`
- **Continue after conflict**: `git cherry-pick --continue`
- **Abort**: `git cherry-pick --abort`

## Worktrees (Multiple Checkouts of Same Repo)
- **List worktrees**: `git worktree list`
- **Add new worktree**: `git worktree add ../featureA featureA`
- **Remove worktree**: `git worktree remove ../featureA`

## Submodules
- **Add submodule**: `git submodule add <url> <path>`
- **Init and update**: `git submodule update --init --recursive`
- **Update submodule to latest**:
```
cd <submodule>
git pull origin main
cd ..
git add <submodule>
```

## Advanced Reset & Recovery
- **Recover deleted commit using reflog**:
```
git reflog
git checkout <lost-commit>
```
- **Interactive staging**: `git add -p`
- **Interactive rebase**:
```
git rebase -i <commit>
```

## Squashing & Cleaning History
- **Squash last N commits**:
```
git rebase -i HEAD~N
# mark all but first as "squash"
```
- **Drop a commit from history**:
  Use interactive rebase and mark commit as `drop`.

## Grepping Inside Repo
- **Search for text in repo**: `git grep "text"`
- **Search only in staged**: `git diff --cached | grep "text"`

## Temporary Work
- **Work-in-progress commit**: `git commit -am "WIP"`
- **Undo WIP commit but keep changes**: `git reset HEAD~1`

## Hooks
- **Location**: `.git/hooks/`
- Example: pre-commit checks, auto-formatters.

---

