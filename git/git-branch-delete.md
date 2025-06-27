# Git Branch Delete

## Purpose
Delete a branch both locally and from remote repository

## Context
Use when a branch is no longer needed and should be completely removed. This command deletes both the local copy and the remote branch. Commonly used after merging feature branches or cleaning up abandoned work.

## Parameters
- `$ARGUMENTS` - Name of the branch to delete
  - Required
  - Example: `feature/old-feature`

## Steps

### 1. Attempt safe deletion
```bash
git branch -d $ARGUMENTS
```
Tries to delete branch safely (only if fully merged). Will fail if branch has unmerged changes.

### 2. Force delete if needed
If step 1 fails with "not fully merged" error:
```bash
git branch -D $ARGUMENTS
```
Forces deletion even with unmerged changes. Use with caution - unmerged changes will be lost.

### 3. Delete remote branch
```bash
git push origin --delete $ARGUMENTS
```
Removes the branch from the remote repository. This permanently deletes the remote branch.

## Validation
- Local branch no longer appears in: `git branch`
- Remote branch no longer appears in: `git branch -r`
- No error messages during deletion

## Error Handling
- **"error: The branch '{branch}' is not fully merged"** - Use -D flag to force delete
- **"error: branch '{branch}' not found"** - Branch doesn't exist locally
- **"remote: error: refusing to delete the current branch"** - Cannot delete the default branch
- **"error: unable to delete '{branch}': remote ref does not exist"** - Branch already deleted on remote
- **"error: Cannot delete branch '{branch}' checked out"** - Switch to another branch first

## Safety Notes
- Force deletion (-D) permanently loses any unmerged changes
- Ensure the branch has been merged or its changes are no longer needed
- Consider creating a backup tag before deleting important branches
- Cannot delete the currently checked out branch

## Examples
- **Delete a merged feature branch**
  ```
  git-branch-delete feature/user-authentication
  ```
  Safely removes the branch from local and remote

- **Force delete an abandoned branch**
  ```
  git-branch-delete experimental/risky-feature
  ```
  Forces deletion even if unmerged, removes from remote