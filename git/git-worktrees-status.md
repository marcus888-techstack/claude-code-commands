# Git Worktrees Status

## Purpose
Show detailed status of all worktrees

## Context
Use to get a comprehensive overview of all worktrees including their branches, uncommitted changes, merge status, and disk usage. Helps identify stale worktrees and track parallel development efforts. Highlights current worktree for context.

## Parameters
- `$ARGUMENTS` - Optional filter pattern
  - Optional
  - Example: `feature/` to show only feature worktrees

## Steps

### 1. Identify current worktree
Marks the worktree you're currently in.

### 2. Process each worktree
```bash
git worktree list --porcelain
```
Extracts detailed information for each worktree.

### 3. Check uncommitted changes
```bash
git status --porcelain | wc -l
```
Counts pending changes in each worktree.

### 4. Check merge status
```bash
git branch --merged develop
git branch --merged main
```
Determines if branches are integrated.

### 5. Apply optional filter
Shows only worktrees matching the pattern.

### 6. Calculate summary statistics
Counts worktrees by type and totals.

### 7. Check for stale worktrees
```bash
git worktree list --porcelain | grep "prunable"
```
Identifies worktrees that can be cleaned.

### 8. Calculate disk usage
Shows total space used by all worktrees.

## Validation
- All worktrees are listed
- Current worktree is highlighted
- Change counts are accurate
- Merge status is correct
- Statistics match worktree count

## Error Handling
- **"not a git repository"** - Not in a git directory
- **Permission errors** - Can't access some worktrees
- **"command not found: du"** - Disk usage unavailable
- **Stale worktree warnings** - Suggests cleanup

## Safety Notes
- Read-only operation
- Safe to run anytime
- Identifies but doesn't modify worktrees
- Helps prevent accumulation of stale worktrees
- Disk usage calculation may be slow

## Examples
- **Show all worktrees status**
  ```
  git-worktrees-status
  ```
  Comprehensive status overview

- **Filter feature worktrees**
  ```
  git-worktrees-status feature/
  ```
  Shows only feature branch status

- **Check specific branch**
  ```
  git-worktrees-status oauth
  ```
  Shows worktrees containing "oauth"