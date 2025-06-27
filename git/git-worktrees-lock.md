# Git Worktrees Lock

## Purpose
Lock or unlock a worktree to prevent accidental removal

## Context
Use to protect important worktrees from being accidentally removed or automatically pruned. Locked worktrees require explicit unlocking before they can be removed. Useful for long-running features or critical development work.

## Parameters
- `$ARGUMENTS` - Action and optional worktree path
  - Format: `<action> [worktree-path]`
  - Actions: `lock`, `unlock`, `status`
  - Example: `lock ../features/critical`

## Steps

### 1. Parse action and path
Extracts the requested action and target worktree.

### 2A. Lock worktree
```bash
git worktree lock "$WORKTREE_PATH"
```
Protects worktree from removal.

### 2B. Unlock worktree
```bash
git worktree unlock "$WORKTREE_PATH"
```
Removes protection from worktree.

### 2C. Show status
```bash
git worktree list --porcelain
```
Displays lock status of all worktrees with visual indicators.

### 3. Verify worktree exists
For lock/unlock actions, confirms worktree is valid.

### 4. Display results
Shows success/failure and current protection status.

## Validation
- Action is valid (lock/unlock/status)
- Worktree exists for lock/unlock
- Lock/unlock operation succeeds
- Status correctly identifies locked worktrees

## Error Handling
- **"Please specify worktree path"** - Missing path for lock/unlock
- **"Worktree not found"** - Invalid worktree path
- **"Failed to lock"** - Already locked or permission issue
- **"Failed to unlock"** - Not locked or permission issue

## Safety Notes
- Locking prevents accidental removal only
- Force removal still possible with --force
- Lock status persists across sessions
- Locked worktrees won't be auto-pruned
- No impact on worktree functionality

## Examples
- **Lock critical feature**
  ```
  git-worktrees-lock lock ../features/critical
  ```
  Protects important development work

- **Unlock completed work**
  ```
  git-worktrees-lock unlock ../features/old-feature
  ```
  Allows normal removal

- **Check all lock status**
  ```
  git-worktrees-lock status
  ```
  Shows protection status with icons