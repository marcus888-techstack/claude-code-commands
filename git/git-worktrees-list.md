# Git Worktrees List

## Purpose
List all Git worktrees with enhanced information

## Context
Use to see all active worktrees in your repository. Shows paths, branches, disk usage, and identifies stale worktrees. Essential for managing multiple parallel development efforts and cleaning up unused worktrees.

## Parameters
- `$ARGUMENTS` - Optional filter pattern
  - Optional
  - Example: `feature/` to show only feature worktrees

## Steps

### 1. Display worktree details
```bash
git worktree list --porcelain
```
Parses and formats worktree information including:
- Path to worktree
- Associated branch (or detached state)
- HEAD commit hash

### 2. Apply optional filter
If filter provided, shows only matching worktrees.

### 3. Generate summary statistics
Counts worktrees by type:
- Total worktrees
- Main/Develop branches
- Feature branches
- Hotfix branches

### 4. Check for stale worktrees
```bash
git worktree list --porcelain | grep -B2 "prunable"
```
Identifies worktrees that can be cleaned up.

### 5. Calculate disk usage
For each worktree, shows disk space used (if `du` command available).

## Validation
- All active worktrees are listed
- Paths are accessible
- Branch associations are correct
- Disk usage is calculated

## Error Handling
- **"not a git repository"** - Not in a git directory
- **"no worktrees found"** - No worktrees configured
- **Permission errors** - Can't access worktree directories

## Safety Notes
- This is a read-only operation
- Safe to run anytime
- Identifies but doesn't remove stale worktrees
- Disk usage may take time for large repos

## Examples
- **List all worktrees**
  ```
  git-worktrees-list
  ```
  Shows comprehensive worktree information

- **Filter feature worktrees**
  ```
  git-worktrees-list feature/
  ```
  Shows only feature branch worktrees

- **Check disk usage**
  ```
  git-worktrees-list
  ```
  Includes size information for each worktree