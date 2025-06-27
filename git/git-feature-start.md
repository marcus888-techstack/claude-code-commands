# Git Feature Start

## Purpose
Start a new feature branch from develop

## Context
Use when beginning work on a new feature. This follows the git-flow branching model where features branch from develop. Ensures you start with the latest code and sets up remote tracking.

## Parameters
- `$ARGUMENTS` - Name for the feature (without feature/ prefix)
  - Required
  - Example: `user-authentication` or `shopping-cart`

## Steps

### 1. Switch to develop branch
```bash
git checkout develop
```
Ensures you're on the develop branch before creating the feature branch.

### 2. Update develop branch
```bash
git pull origin develop
```
Gets the latest changes from remote develop to ensure you start with current code.

### 3. Create feature branch
```bash
git checkout -b feature/$ARGUMENTS
```
Creates a new branch with the feature/ prefix for organization.

### 4. Push to remote with tracking
```bash
git push -u origin feature/$ARGUMENTS
```
Creates the branch on remote and sets up tracking with -u flag.

## Validation
- You're now on the feature branch
- Branch appears in `git branch` with feature/ prefix
- Remote tracking is set up (visible in `git branch -vv`)
- Push/pull works without specifying remote/branch

## Error Handling
- **"fatal: A branch named 'feature/X' already exists"** - Feature branch already exists, use different name or delete old branch
- **"error: pathspec 'develop' did not match"** - Develop branch doesn't exist, create it first
- **"fatal: Couldn't find remote ref develop"** - Develop doesn't exist on remote, push it first
- **Connection errors** - Check network and repository access

## Safety Notes
- Always ensure develop is up-to-date before branching
- Use descriptive feature names for clarity
- Consider team naming conventions for features

## Examples
- **Start a user authentication feature**
  ```
  git-feature-start user-authentication
  ```
  Creates feature/user-authentication branch

- **Start a payment integration feature**
  ```
  git-feature-start payment-gateway
  ```
  Creates feature/payment-gateway branch