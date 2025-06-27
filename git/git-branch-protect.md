# Git Branch Protect

## Purpose
Set local branch protection configuration

## Context
Configures git to mark branches as protected locally. This is a safety measure that works with git hooks and tools. For full protection, you must also configure your git hosting service.

## Parameters
- `$ARGUMENTS` - Specific branch to protect
  - Optional: if not provided, protects main/master/develop
  - Example: `staging`

## Steps

### 1. Determine branches to protect
Check if `$ARGUMENTS` is provided:
- If yes: Protect only the specified branch
- If no: Protect default branches (main, master, develop)

### 2. Set protection config
If no specific branch provided:
```bash
git config branch.main.protect true
git config branch.master.protect true
git config branch.develop.protect true
```

If specific branch provided:
```bash
git config branch.$ARGUMENTS.protect true
```

These mark branches as protected in git config, which works with git hooks that check this setting.

### 3. Display protection status
Show which branches were protected and remind about remote protection requirements.

## Validation
- Config values are set: `git config --get branch.{branch}.protect`
- Protected branches show "true" value
- No error messages during configuration

## Error Handling
- **"error: key does not contain a section"** - Invalid branch name format
- **"fatal: not in a git directory"** - Must be run inside a git repository
- **Branch doesn't exist warning** - Config is set but branch may not exist yet

## Safety Notes
- Local protection only prevents some operations with compatible tools
- For full protection, configure your git hosting service (GitHub/GitLab/Bitbucket)
- Protection rules vary by platform and must be set there

## Examples
- **Protect default branches**
  ```
  git-branch-protect
  ```
  Protects main, master, and develop branches locally

- **Protect a staging branch**
  ```
  git-branch-protect staging
  ```
  Marks staging branch as protected in local config

- **Protect a release branch**
  ```
  git-branch-protect release/v2.0
  ```
  Marks release/v2.0 as protected locally