# Git Branch Sync

## Purpose
Sync branch with remote upstream

## Context
Use to update a local branch with the latest changes from the remote repository. Essential before starting new work or resolving conflicts. Ensures your branch is up-to-date with the remote.

## Parameters
- `$ARGUMENTS` - Name of the branch to sync
  - Required
  - Example: `develop` or `feature/new-feature`

## Steps

### 1. Fetch latest changes
```bash
git fetch origin
```
Downloads all new commits and branches from the remote repository without merging them.

### 2. Checkout the branch
```bash
git checkout $ARGUMENTS
```
Switches to the branch you want to sync.

### 3. Pull latest changes
```bash
git pull origin $ARGUMENTS
```
Merges the remote changes into your local branch.

### 4. Show current status
```bash
git status
```
Displays the current state of your working directory and staging area.

### 5. Show recent commits
```bash
git log --oneline -5
```
Shows the last 5 commits to verify the sync was successful.

## Validation
- No merge conflicts reported
- git status shows "Your branch is up to date"
- Recent commits include expected changes from remote

## Error Handling
- **"error: pathspec '{branch}' did not match"** - Branch doesn't exist locally, use git checkout -b {branch} origin/{branch}
- **"CONFLICT (content): Merge conflict"** - Resolve conflicts manually, then commit
- **"fatal: couldn't find remote ref"** - Branch doesn't exist on remote
- **"error: Your local changes would be overwritten"** - Stash or commit local changes first

## Safety Notes
- Always commit or stash local changes before syncing
- If conflicts occur, resolve them carefully to avoid losing work
- Consider creating a backup branch before syncing if you have important uncommitted work

## Examples
- **Sync develop branch**
  ```
  git-branch-sync develop
  ```
  Updates local develop branch with remote changes

- **Sync a feature branch**
  ```
  git-branch-sync feature/user-profile
  ```
  Updates your feature branch with latest remote changes