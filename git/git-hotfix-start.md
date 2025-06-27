# Git Hotfix Start

## Purpose
Start a hotfix branch from master for urgent production fixes

## Context
Use when a critical bug is found in production that needs immediate attention. Hotfixes branch from master (production code) rather than develop, allowing urgent fixes without including unreleased features.

## Parameters
- `$ARGUMENTS` - Name for the hotfix (without hotfix/ prefix)
  - Required
  - Example: `security-patch` or `critical-bug-fix`

## Steps

### 1. Switch to master branch
```bash
git checkout master
```
Ensures you're on the master/main branch (production code).

### 2. Update master branch
```bash
git pull origin master
```
Gets the latest production code to ensure fix is based on current version.

### 3. Create hotfix branch
```bash
git checkout -b hotfix/$ARGUMENTS
```
Creates a new branch with hotfix/ prefix for urgent fixes.

### 4. Push to remote with tracking
```bash
git push -u origin hotfix/$ARGUMENTS
```
Creates the branch on remote and sets up tracking.

## Validation
- You're now on the hotfix branch
- Branch appears in `git branch` with hotfix/ prefix
- Remote tracking is set up
- Branch is based on latest master, not develop

## Error Handling
- **"fatal: A branch named 'hotfix/X' already exists"** - Hotfix branch exists, use different name
- **"error: pathspec 'master' did not match"** - Master branch doesn't exist, check for 'main' instead
- **"fatal: Couldn't find remote ref master"** - Master doesn't exist on remote
- **Connection errors** - Check network and repository access

## Safety Notes
- Only use for critical production issues
- Keep hotfixes minimal and focused
- Test thoroughly despite urgency
- Plan to merge to both master AND develop

## Examples
- **Start a security hotfix**
  ```
  git-hotfix-start security-vulnerability
  ```
  Creates hotfix/security-vulnerability branch

- **Start a critical bug fix**
  ```
  git-hotfix-start payment-processing-error
  ```
  Creates hotfix/payment-processing-error branch