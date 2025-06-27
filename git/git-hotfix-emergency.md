# Git Hotfix Emergency

## Purpose
Fast-track critical hotfix deployment with immediate push to production

## Context
Use for critical production issues requiring immediate attention. This creates a hotfix branch and provides clear instructions for rapid deployment. Only use when standard procedures are too slow for the severity of the issue.

## Parameters
- `$ARGUMENTS` - Name for the emergency hotfix (without hotfix/ prefix)
  - Required
  - Example: `critical-security-breach` or `data-loss-prevention`

## Steps

### 1. Switch to main/master branch
```bash
git checkout main
```
Moves to the production branch.

### 2. Update main branch
```bash
git pull origin main
```
Ensures you have the latest production code.

### 3. Create emergency hotfix branch
```bash
git checkout -b hotfix/$ARGUMENTS
```
Creates the hotfix branch for urgent changes.

### 4. Display emergency instructions
Shows clear steps for the developer to follow:
- Make critical fixes
- Test thoroughly (even in emergencies)
- Complete with hotfix finish command
- Explains the automated deployment process

## Validation
- Hotfix branch is created from latest main
- Developer receives clear emergency procedures
- Branch follows hotfix naming convention

## Error Handling
- **"error: pathspec 'main' did not match"** - Try 'master' instead
- **"fatal: couldn't find remote ref"** - Check branch name and remote
- **"already exists"** - Hotfix branch exists, check if work is in progress

## Safety Notes
- EMERGENCY USE ONLY - bypasses normal review processes
- Document all changes thoroughly
- Test as much as time allows
- Notify team immediately
- Follow up with proper code review after deployment
- Consider rollback plan before starting

## Examples
- **Critical security breach**
  ```
  git-hotfix-emergency security-breach-fix
  ```
  Creates hotfix/security-breach-fix for immediate action

- **Production data corruption**
  ```
  git-hotfix-emergency data-corruption-fix
  ```
  Starts emergency fix for data issues