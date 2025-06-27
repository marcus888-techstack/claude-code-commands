# Git Feature Update

## Purpose
Update feature branch with latest changes from develop

## Context
Use to keep your feature branch up-to-date with the latest changes from develop. This prevents large merge conflicts later and ensures you're working with current code. Should be done regularly during long-running features.

## Parameters
- `$ARGUMENTS` - Name of the feature (without feature/ prefix)
  - Required
  - Example: `user-authentication`

## Steps

### 1. Switch to develop branch
```bash
git checkout develop
```
Move to the develop branch to get latest changes.

### 2. Update develop from remote
```bash
git pull origin develop
```
Fetches and merges the latest changes from remote develop.

### 3. Switch to feature branch
```bash
git checkout feature/$ARGUMENTS
```
Returns to your feature branch.

### 4. Merge develop into feature
```bash
git merge develop
```
Brings all the latest develop changes into your feature branch.

### 5. Push updated feature branch
```bash
git push origin feature/$ARGUMENTS
```
Updates the remote feature branch with the merged changes.

## Validation
- No merge conflicts (or conflicts resolved)
- Feature branch contains latest develop commits
- Tests still pass after merge
- Feature functionality still works

## Error Handling
- **"error: pathspec 'feature/X' did not match"** - Feature branch doesn't exist
- **"CONFLICT (content): Merge conflict"** - Resolve conflicts manually, then commit
- **"Already up to date"** - Feature branch already has latest changes
- **"fatal: Not possible to fast-forward"** - Complex history, may need to resolve manually

## Safety Notes
- Always commit or stash local changes before updating
- Test thoroughly after merging develop changes
- Resolve conflicts carefully to avoid breaking features
- Consider rebasing for cleaner history (if team agrees)

## Examples
- **Update authentication feature**
  ```
  git-feature-update user-authentication
  ```
  Merges latest develop into feature/user-authentication

- **Update payment feature**
  ```
  git-feature-update payment-gateway
  ```
  Updates feature/payment-gateway with develop changes