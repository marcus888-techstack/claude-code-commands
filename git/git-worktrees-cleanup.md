# Git Worktrees Cleanup

## Purpose
Clean up worktrees for merged branches

## Context
Use to automatically remove worktrees whose branches have been merged to main or develop. Helps maintain a clean workspace by removing completed work. Includes safety checks for uncommitted changes and option to filter by branch pattern.

## Parameters
- `$ARGUMENTS` - Optional branch pattern filter
  - Optional
  - Example: `feature/` to cleanup only feature worktrees

## Steps

### 1. List all worktrees
```bash
git worktree list --porcelain
```
Gets comprehensive worktree information.

### 2. Skip permanent branches
Excludes main and develop worktrees from cleanup.

### 3. Apply optional filter
If provided, only processes worktrees matching the pattern.

### 4. Check merge status
```bash
git branch --merged develop
git branch --merged main
```
Determines if branch has been integrated.

### 5. Verify uncommitted changes
```bash
git -C "$WORKTREE" status --porcelain
```
Prevents loss of uncommitted work.

### 6. Remove merged worktrees
```bash
git worktree remove "$WORKTREE"
```
Removes worktree directory and metadata.

### 7. Delete merged branches
```bash
git branch -d "$BRANCH"
git push origin --delete "$BRANCH"
```
Cleans up local and remote branches.

### 8. Prune stale entries
```bash
git worktree prune
```
Removes references to deleted worktrees.

### 9. Display summary
Shows cleanup statistics and remaining worktrees.

## Validation
- Only merged branches are removed
- Uncommitted changes are preserved
- Worktree removal is successful
- Branch deletion completes
- Stale entries are pruned

## Error Handling
- **"has uncommitted changes"** - Skips worktree with unsaved work
- **"not merged"** - Branch not integrated to main/develop
- **"failed to remove"** - Permission or access issues
- **Remote deletion fails** - Remote branch may be protected

## Safety Notes
- Never removes main or develop worktrees
- Checks for uncommitted changes before removal
- Only deletes merged branches
- Remote deletion may fail silently
- Use --force flag cautiously

## Examples
- **Clean all merged worktrees**
  ```
  git-worktrees-cleanup
  ```
  Removes all merged feature/hotfix worktrees

- **Clean only feature worktrees**
  ```
  git-worktrees-cleanup feature/
  ```
  Targets feature branches only

- **Clean release worktrees**
  ```
  git-worktrees-cleanup release/
  ```
  Removes completed release worktrees