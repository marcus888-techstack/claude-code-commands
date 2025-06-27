# Git Workflow Status

## Purpose
Show comprehensive workflow status including branches, versions, and pending work

## Context
Use to get a complete overview of your repository's git-flow status. Shows current branch, latest versions, active feature/hotfix branches, uncommitted changes, and recent activity. Essential for understanding project state.

## Parameters
- None required

## Steps

### 1. Show current branch
```bash
git branch --show-current
```
Displays which branch you're currently on.

### 2. Display latest versions
```bash
echo "Main: $(git describe --tags --abbrev=0 main 2>/dev/null || echo 'No tags')"
echo "Develop: $(git describe --tags --abbrev=0 develop 2>/dev/null || echo 'No tags')"
```
Shows the most recent tags on main branches.

### 3. List active branches
```bash
git branch -a | grep -E "(feature/|release/|hotfix/)" | sed 's/remotes\/origin\///'
```
Finds all feature, release, and hotfix branches.

### 4. Check working directory
```bash
git status -s || echo "✓ Clean"
```
Shows uncommitted changes or confirms clean state.

### 5. Find branches ready to merge
```bash
git branch --no-merged develop | grep -E "(feature/)"
```
Lists feature branches not yet merged to develop.

### 6. Show recent activity
```bash
git log --oneline --graph --decorate -10
```
Displays the last 10 commits with branch visualization.

## Validation
- All sections display correctly
- Version tags are found if they exist
- Active branches are listed
- Working directory status is accurate

## Error Handling
- **"No tags"** - Repository has no version tags yet
- **"No branches found"** - No active feature/hotfix branches
- **"fatal: branch does not exist"** - Missing main or develop branch

## Safety Notes
- This is a read-only operation
- Safe to run anytime
- No changes are made to repository

## Examples
- **Check workflow status**
  ```
  git-workflow-status
  ```
  Shows complete repository workflow state

- **Before starting new work**
  ```
  git-workflow-status
  ```
  Helps decide what to work on next

- **During team standup**
  ```
  git-workflow-status
  ```
  Provides overview for discussion