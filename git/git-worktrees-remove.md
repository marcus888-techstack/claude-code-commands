# Git Worktrees Remove

## Purpose
Remove a Git worktree safely

## Context
Use to clean up worktrees when done with parallel development. Checks for uncommitted changes and unmerged branches before removal. Optionally deletes the associated branch if it's been merged.

## Parameters
- `$ARGUMENTS` - Path to the worktree to remove
  - Required
  - Example: `../features/oauth`

## Steps

### 1. Verify worktree exists
```bash
git worktree list | grep "$WORKTREE_PATH"
```
Ensures the specified worktree is valid.

### 2. Get associated branch
Extracts branch name from worktree metadata.

### 3. Check for uncommitted changes
```bash
cd "$WORKTREE_PATH" && git status --porcelain
```
Prevents accidental loss of work.

### 4. Check if branch is merged
Verifies if branch has been merged to develop or main.

### 5. Remove worktree
```bash
git worktree remove "$WORKTREE_PATH"
```
Removes the worktree directory and metadata.

### 6. Offer to delete merged branch
If branch is merged, prompts to delete it.

### 7. Show remaining worktrees
Lists worktrees still active.

## Validation
- Worktree is successfully removed
- No uncommitted changes are lost
- Branch deletion is intentional
- Remaining worktrees are listed

## Error Handling
- **"Worktree not found"** - Invalid path specified
- **"has uncommitted changes"** - Save or stash changes first
- **"branch is not merged"** - Warns before removing unmerged work
- **"failed to remove"** - May need --force flag

## Safety Notes
- Always check for uncommitted changes
- Verify branch merge status
- Use --force only when certain
- Worktree removal is permanent
- Branch can be preserved even if worktree is removed

## Examples
- **Remove completed feature worktree**
  ```
  git-worktrees-remove ../features/oauth
  ```
  Removes worktree after feature completion

- **Force remove with changes**
  ```
  git worktree remove --force ../experiments/test
  ```
  Removes worktree losing uncommitted changes

- **Remove and delete branch**
  ```
  git-worktrees-remove ../features/old-feature
  ```
  Prompts to delete merged branch