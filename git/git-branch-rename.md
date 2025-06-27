# Git Branch Rename

## Purpose
Rename a branch locally and update remote

## Context
Use when you need to change a branch name to better reflect its purpose or to follow naming conventions. This updates both the local branch name and the remote repository.

## Parameters
- `$ARGUMENTS` - Old and new branch names separated by space
  - Required
  - Format: `old-name new-name`
  - Example: `feature/login feature/authentication`

## Steps

### 1. Parse arguments
```bash
OLD_NAME=$(echo $ARGUMENTS | cut -d' ' -f1)
NEW_NAME=$(echo $ARGUMENTS | cut -d' ' -f2)
```
Extracts the old and new branch names from the arguments.

### 2. Rename local branch
```bash
git branch -m $OLD_NAME $NEW_NAME
```
Renames the branch locally. The -m flag moves/renames the branch.

### 3. Delete old remote branch
```bash
git push origin --delete $OLD_NAME
```
Removes the old branch name from the remote repository.

### 4. Push new branch to remote
```bash
git push origin -u $NEW_NAME
```
Creates the new branch on remote and sets up tracking with -u flag.

## Validation
- Old branch no longer exists locally or remotely
- New branch exists with the same commit history
- Upstream tracking is properly configured

## Error Handling
- **"error: refname refs/heads/{old} not found"** - Old branch doesn't exist
- **"error: branch '{new}' already exists"** - New branch name is already taken
- **"error: unable to delete '{old}': remote ref does not exist"** - Old branch already deleted on remote
- **Missing second argument** - Both old and new names are required

## Safety Notes
- Ensure no one else is working on the branch before renaming
- Update any open pull requests to reference the new branch name
- Update any CI/CD configurations that reference the old branch name

## Examples
- **Rename a feature branch**
  ```
  git-branch-rename feature/login feature/user-authentication
  ```
  Changes branch name from feature/login to feature/user-authentication

- **Fix a typo in branch name**
  ```
  git-branch-rename feature/serach-bar feature/search-bar
  ```
  Corrects the spelling in the branch name