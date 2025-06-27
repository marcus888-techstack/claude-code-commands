# Git Workflow Sync

## Purpose
Synchronize branches after tag-based releases or hotfixes

## Context
Use to keep develop and main branches in sync after releases or hotfixes. This ensures develop includes all production changes and prevents divergence between branches. Critical for maintaining a clean git-flow.

## Parameters
- `$ARGUMENTS` - Sync operation type
  - Required
  - Options: `release`, `hotfix`, or `status`
  - Example: `release`

## Steps

### 1. Status check operation
If `status` is specified:
- Shows current versions on both branches
- Calculates commits ahead/behind
- Recommends sync if needed

### 2. Release sync operation
If `release` is specified:
- Updates both main and develop
- Merges main into develop if needed
- Creates development tag for the release version
- Ensures develop has all release changes

### 3. Hotfix sync operation
If `hotfix` is specified:
- Merges hotfix from main into develop
- Creates development tag for hotfix version
- Ensures critical fixes are in both branches

### 4. Validation and confirmation
Shows sync results and current branch state.

## Validation
- Branches are properly synchronized
- Development tags are created
- No divergence between branches
- Merge commits preserve history

## Error Handling
- **"Unknown sync type"** - Use release, hotfix, or status
- **"Merge conflicts"** - Resolve manually then commit
- **"Branch not found"** - Ensure main and develop exist
- **"Already synchronized"** - No action needed

## Safety Notes
- Always sync after releases and hotfixes
- Resolve conflicts carefully
- Test after synchronization
- Maintains branch history with --no-ff

## Examples
- **Check sync status**
  ```
  git-workflow-sync status
  ```
  Shows if branches need synchronization

- **After a release**
  ```
  git-workflow-sync release
  ```
  Syncs develop with the released version

- **After a hotfix**
  ```
  git-workflow-sync hotfix
  ```
  Ensures hotfix is in develop branch