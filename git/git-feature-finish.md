# Git Feature Finish

## Purpose
Complete feature development and create a pull request for review

## Context
Use when a feature is complete and ready for review. This command pushes the feature branch and creates a pull request to develop branch. Follows git-flow conventions with code review process.

## Parameters
- `$ARGUMENTS` - Name of the feature to finish (without feature/ prefix)
  - Required
  - Example: `user-authentication`

## Steps

### 1. Validate inputs
Check that feature name is provided and the feature branch exists.

### 2. Update feature branch
```bash
git checkout feature/$FEATURE_NAME
git pull origin develop
```
Ensures feature branch has the latest changes from develop.

### 3. Push feature branch
```bash
git push origin feature/$FEATURE_NAME
```
Pushes the feature branch to remote repository.

### 4. Create pull request
```bash
gh pr create --base develop --head feature/$FEATURE_NAME \
  --title "Feature: $FEATURE_NAME" \
  --body "## Summary
- Completes feature: $FEATURE_NAME

## Changes
- [Add description of changes]

## Testing
- [Add testing instructions]"
```
Creates a pull request using GitHub CLI.

### 5. Open pull request in browser
```bash
gh pr view --web
```
Opens the created pull request in the default web browser for review.

### 6. Switch back to develop
```bash
git checkout develop
```
Returns to develop branch after creating PR.

### 7. Note on cleanup
Feature branch cleanup (deletion) should happen after PR is merged:
- Use GitHub's "Delete branch" button after merge
- Or run `git push origin --delete feature/$FEATURE_NAME` manually
- Worktree removal: `git worktree remove ../features/$FEATURE_NAME`

## Validation
- Feature branch is pushed to remote
- Pull request is created successfully
- PR link is displayed for review
- Feature branch remains available for additional commits

## Error Handling
- **"Branch 'feature/X' not found"** - Feature branch doesn't exist
- **"gh: command not found"** - Install GitHub CLI: `brew install gh`
- **"Pull request already exists"** - Update existing PR with `gh pr view`
- **"Authentication failed"** - Run `gh auth login` to authenticate

## Safety Notes
- Always test feature thoroughly before creating PR
- Ensure CI/CD passes on feature branch
- Add reviewers to the PR after creation
- Update PR description with detailed changes
- Wait for approvals before merging

## Examples
- **Finish a user authentication feature**
  ```
  git-feature-finish user-authentication
  ```
  Creates PR for feature/user-authentication to develop

- **Finish a payment feature**
  ```
  git-feature-finish payment-integration
  ```
  Creates PR for the payment integration feature

## Prerequisites
- GitHub CLI installed (`gh`)
- Authenticated with GitHub (`gh auth login`)
- Repository has GitHub remote configured