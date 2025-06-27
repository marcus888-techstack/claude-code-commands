# Git Branch Cleanup

## Purpose
Clean up merged branches while preserving protected branches

## Context
Use this command after features have been merged to keep your repository tidy. It removes local branches that have been fully merged while protecting important branches. Also cleans up stale remote tracking references.

## Parameters
- `$ARGUMENTS` - Additional branch names to protect from deletion (besides master/main/develop)
  - Optional: defaults to none
  - Example: `staging|production`

## Steps

### 1. List merged branches
```bash
git branch --merged
```
Shows all branches that have been fully merged into the current branch. This ensures we only delete branches whose changes are preserved.

### 2. Filter protected branches
```bash
grep -v -E "master|main|develop|$ARGUMENTS"
```
Excludes branches that should never be deleted. Always protects master, main, and develop by default.

### 3. Delete filtered branches
```bash
xargs -n 1 git branch -d
```
Deletes each merged branch safely. Using -d (not -D) ensures only fully merged branches are deleted.

### 4. Prune remote tracking
```bash
git remote prune origin
```
Removes local references to deleted remote branches. Keeps your remote tracking branches in sync.

### 5. Display remaining branches
```bash
git branch -av
```
Shows all remaining local and remote branches. Helps verify the cleanup was successful.

## Validation
- Protected branches (master/main/develop) still exist
- Only merged branches were deleted
- Remote tracking branches are synchronized
- No error messages about unmerged branches

## Error Handling
- **"error: The branch 'X' is not fully merged"** - Branch has unmerged changes, use -D flag only if you're sure
- **"fatal: branch name required"** - No branches to delete, repository is already clean
- **"grep: no pattern"** - No merged branches found matching criteria

## Safety Notes
- This command only deletes fully merged branches with -d flag
- Always verify important branches are protected before running
- Consider creating a backup tag before large cleanups

## Examples
- **Basic cleanup keeping only default protected branches**
  ```
  git-branch-cleanup
  ```
  Removes all merged feature branches except master/main/develop

- **Cleanup with additional protected branches**
  ```
  git-branch-cleanup "staging|production"
  ```
  Keeps master/main/develop plus staging and production branches