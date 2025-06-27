# Git Merge No-FF

## Purpose
Merge branch with --no-ff to preserve branch history

## Context
Use when you want to maintain a clear history of feature branches. The --no-ff flag creates a merge commit even when a fast-forward merge is possible, making it easier to see where features were developed and merged.

## Parameters
- `$ARGUMENTS` - Name of the branch to merge
  - Required
  - Example: `feature/user-auth` or `hotfix/security-fix`

## Steps

### 1. Validate input
Check that a branch name is provided.

### 2. Get current branch
```bash
git branch --show-current
```
Identifies which branch you're merging into.

### 3. Perform no-fast-forward merge
```bash
git merge --no-ff $BRANCH -m "Merge branch '$BRANCH' into $CURRENT

This merge preserves the branch history and creates a merge commit
even if a fast-forward merge is possible."
```
Creates a merge commit with descriptive message.

### 4. Display merge result
```bash
git log -1 --oneline
```
Shows the created merge commit.

## Validation
- Merge commit is created (visible in git log)
- Target branch changes are integrated
- Branch history is preserved
- No unresolved conflicts

## Error Handling
- **"Please specify branch to merge"** - Branch name is required
- **"merge: {branch} - not something we can merge"** - Branch doesn't exist
- **"CONFLICT (content)"** - Resolve conflicts before completing merge
- **"Already up to date"** - No changes to merge from branch

## Safety Notes
- Resolve all conflicts before committing
- Review changes before merging
- Consider using pull requests for code review
- The merge commit helps maintain clear project history

## Examples
- **Merge a feature branch**
  ```
  git-merge-no-ff feature/shopping-cart
  ```
  Creates merge commit for shopping cart feature

- **Merge a hotfix**
  ```
  git-merge-no-ff hotfix/security-patch
  ```
  Preserves hotfix history with merge commit