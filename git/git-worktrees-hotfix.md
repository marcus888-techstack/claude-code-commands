# Git Worktrees Hotfix

## Purpose
Create a hotfix branch worktree for urgent production fixes

## Context
Use for critical production issues that can't wait for the normal release cycle. Creates a hotfix branch from main (production) and sets up a dedicated workspace. Includes tracking file and ensures fix is based on current production version.

## Parameters
- `$ARGUMENTS` - Hotfix name
  - Required
  - Example: `security-patch`, `critical-bug-fix`, `cve-2024-001`

## Steps

### 1. Validate hotfix name
Ensures a descriptive hotfix name is provided.

### 2. Update main branch
```bash
git checkout main
git pull origin main
```
Ensures hotfix starts from latest production code.

### 3. Get production version
```bash
git describe --tags --abbrev=0 main
```
Identifies current production version for tracking.

### 4. Create branch name
Formats as `hotfix/<hotfix-name>`.

### 5. Check branch availability
Prevents duplicate hotfix branches.

### 6. Create hotfix worktree
```bash
git worktree add -b "$BRANCH_NAME" "$WORKTREE_PATH"
```
Creates new hotfix branch with dedicated workspace.

### 7. Set up hotfix tracking
Creates `.hotfix-info` file with:
- Hotfix details and base version
- Issue description placeholder
- Solution approach section
- Testing checklist

### 8. Display hotfix workflow
Shows critical steps for hotfix completion.

## Validation
- Hotfix name is provided
- Main branch is up to date
- Branch doesn't already exist
- Worktree is created successfully
- Tracking file is created

## Error Handling
- **"Usage: /git/worktrees/hotfix"** - Missing hotfix name
- **"Branch already exists"** - Duplicate hotfix attempt
- **"failed to pull"** - Network or authentication issues
- **"No tags found"** - Repository has no version tags

## Safety Notes
- Always starts from production (main)
- Prevents duplicate hotfixes
- Tracks base version for patches
- Separate workspace for isolation
- Follows emergency fix workflow

## Examples
- **Security vulnerability fix**
  ```
  git-worktrees-hotfix security-patch
  ```
  Creates hotfix/security-patch worktree

- **Critical bug fix**
  ```
  git-worktrees-hotfix critical-bug-fix
  ```
  Creates urgent fix workspace

- **CVE response**
  ```
  git-worktrees-hotfix cve-2024-001
  ```
  Creates CVE-specific hotfix branch