# Git Worktrees Switch

## Purpose
Switch between Git worktrees quickly

## Context
Use to navigate between different worktrees with smart path resolution. Searches by name, branch, or path. Shows available worktrees when called without arguments. Opens a new shell in the target worktree to maintain context.

## Parameters
- `$ARGUMENTS` - Target worktree name, branch, or path
  - Optional: shows list when omitted
  - Example: `develop`, `feature/oauth`, `../features/payment`

## Steps

### 1. Show available worktrees
If no target specified:
```bash
git worktree list --porcelain
```
Displays formatted list with current worktree marked.

### 2. Find target worktree
Searches in order:
- Direct path match
- Common locations (../features, ../hotfixes)
- Branch name match

### 3. Verify worktree exists
Confirms the found path is valid.

### 4. Change to worktree
```bash
cd "$WORKTREE_PATH"
```
Navigates to the worktree directory.

### 5. Show worktree status
```bash
git status -sb
```
Displays branch and change summary.

### 6. Start new shell
```bash
exec $SHELL
```
Opens fresh shell in worktree context.

## Validation
- Target worktree is found
- Path exists and is accessible
- Git recognizes the worktree
- Status is displayed correctly
- New shell starts in worktree

## Error Handling
- **"Worktree not found"** - Invalid target specified
- **No arguments** - Shows available worktrees
- **Multiple matches** - Uses first found
- **Permission denied** - Can't access worktree

## Safety Notes
- Read-only search operation
- Preserves current shell context
- New shell inherits environment
- Exit new shell to return
- No modifications to worktrees

## Examples
- **Switch to develop**
  ```
  git-worktrees-switch develop
  ```
  Quick switch to development branch

- **Switch to feature**
  ```
  git-worktrees-switch feature/oauth
  ```
  Uses branch name to find worktree

- **List available worktrees**
  ```
  git-worktrees-switch
  ```
  Shows all worktrees with current marked

- **Switch by path**
  ```
  git-worktrees-switch ../hotfixes/security
  ```
  Direct path navigation