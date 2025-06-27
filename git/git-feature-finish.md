# Git Feature Finish

## Purpose
Complete feature and merge to develop with automatic version tagging

## Context
Use when a feature is complete and ready to be integrated into develop. This command merges the feature branch, creates a development version tag, and cleans up the branch. Follows git-flow conventions.

## Parameters
- `$ARGUMENTS` - Name of the feature to finish (without feature/ prefix)
  - Required
  - Example: `user-authentication`

## Steps

### 1. Validate inputs
Check that feature name is provided and the feature branch exists.

### 2. Update develop branch
```bash
git checkout develop
git pull origin develop
```
Ensures develop has the latest changes before merging.

### 3. Determine new version
- Get current version from develop branch tags
- Parse version components (major.minor.patch)
- Increment patch version
- Add _dev suffix for development version

### 4. Merge feature branch
```bash
git merge --no-ff feature/$FEATURE_NAME -m "Merge feature/$FEATURE_NAME into develop

Feature: $FEATURE_NAME completed"
```
Uses --no-ff to preserve feature branch history.

### 5. Push changes and tag
```bash
git push origin develop
git tag -a "v{MAJOR}.{MINOR}.{PATCH}_dev" -m "Feature merged: $FEATURE_NAME"
git push origin "v{MAJOR}.{MINOR}.{PATCH}_dev"
```
Pushes the merge and creates a development version tag.

### 6. Clean up branches
```bash
git branch -d feature/$FEATURE_NAME
git push origin --delete feature/$FEATURE_NAME
```
Removes local and remote feature branches.

### 7. Remove worktree (if exists)
```bash
git worktree remove ../features/$FEATURE_NAME
```
Cleans up any associated worktree.

## Validation
- Feature branch is merged into develop
- New development version tag is created
- Feature branch is deleted locally and remotely
- No merge conflicts remain

## Error Handling
- **"Branch 'feature/X' not found"** - Feature branch doesn't exist
- **"No version tags found on develop"** - Uses v0.0.0 as starting version
- **"Merge failed"** - Resolve conflicts manually, then retry
- **"Cannot delete branch"** - Branch might not be fully merged

## Safety Notes
- Always test feature thoroughly before finishing
- Ensure CI/CD passes on feature branch
- Communicate with team before merging large features
- The --no-ff flag preserves feature history

## Examples
- **Finish a user authentication feature**
  ```
  git-feature-finish user-authentication
  ```
  Merges feature/user-authentication to develop and tags it

- **Finish a payment feature**
  ```
  git-feature-finish payment-integration
  ```
  Completes the payment integration feature