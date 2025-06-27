# Git Hotfix Finish

## Purpose
Complete hotfix by merging to master and develop

## Context
Use when a hotfix is tested and ready for production. This merges the fix to both master (for immediate release) and develop (to include in future releases). Creates a tag for tracking the hotfix.

## Parameters
- `$ARGUMENTS` - Name of the hotfix to finish (without hotfix/ prefix)
  - Required
  - Example: `security-patch`

## Steps

### 1. Merge to master branch
```bash
git checkout master
git pull origin master
git merge --no-ff hotfix/$ARGUMENTS
```
Updates master and merges the hotfix with --no-ff to preserve history.

### 2. Tag the hotfix
```bash
git tag -a "hotfix-$ARGUMENTS" -m "Hotfix: $ARGUMENTS"
git push origin master
git push origin "hotfix-$ARGUMENTS"
```
Creates a tag to mark the hotfix release and pushes everything.

### 3. Merge to develop branch
```bash
git checkout develop
git pull origin develop
git merge --no-ff hotfix/$ARGUMENTS
git push origin develop
```
Ensures the fix is also included in ongoing development.

### 4. Clean up branches
```bash
git branch -d hotfix/$ARGUMENTS
git push origin --delete hotfix/$ARGUMENTS
```
Removes the hotfix branch locally and remotely.

## Validation
- Hotfix is merged to both master and develop
- Tag is created and pushed
- Hotfix branch is deleted
- No merge conflicts remain

## Error Handling
- **"branch 'hotfix/X' not found"** - Hotfix branch doesn't exist
- **"CONFLICT (content)"** - Resolve merge conflicts before continuing
- **"tag already exists"** - Choose a different tag name
- **"failed to push"** - Check permissions and network

## Safety Notes
- Test the hotfix thoroughly before finishing
- Ensure CI/CD passes on the hotfix branch
- Coordinate deployment immediately after merging to master
- Monitor production after deployment

## Examples
- **Finish a security hotfix**
  ```
  git-hotfix-finish security-vulnerability
  ```
  Merges to master and develop, tags as hotfix-security-vulnerability

- **Finish a payment bug fix**
  ```
  git-hotfix-finish payment-processing-error
  ```
  Completes the payment processing hotfix