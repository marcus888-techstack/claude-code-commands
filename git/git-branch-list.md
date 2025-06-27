# Git Branch List

## Purpose
List branches with detailed information and optional filtering

## Context
Use to get an overview of all branches in your repository. Shows both local and remote branches with their latest commits. Helpful for understanding branch structure and finding specific branches.

## Parameters
- `$ARGUMENTS` - Optional pattern to filter branch names
  - Optional
  - Example: `feature/`

## Steps

### 1. Display header
```bash
echo "=== Git Branches ==="
```

### 2. Show current branch
```bash
git branch --show-current
```
Identifies which branch you're currently on. Important context for branch operations.

### 3. List local branches
```bash
git branch -vv
```
Shows local branches with commit info and tracking. The -vv flag shows upstream branch relationships.
If `$ARGUMENTS` is provided, filter with: `grep $ARGUMENTS`

### 4. List remote branches
```bash
git branch -r -v
```
Shows all remote branches with latest commits. Helps identify branches available for checkout.
If `$ARGUMENTS` is provided, filter with: `grep $ARGUMENTS`

### 5. Generate summary statistics
- Count total local branches: `git branch | wc -l`
- Count total remote branches: `git branch -r | wc -l`
- Count feature branches: `git branch | grep -c "feature/"`
- Count release branches: `git branch | grep -c "release/"`
- Count hotfix branches: `git branch | grep -c "hotfix/"`

## Validation
- Current branch is displayed
- Local and remote branches are listed
- Summary counts are reasonable (non-negative)
- If filter used, only matching branches shown

## Error Handling
- **"fatal: not a git repository"** - Not in a git directory
- **"grep: no matches found"** - No branches match the filter pattern
- **Empty branch list** - Repository might be new with no branches

## Safety Notes
- This is a read-only operation, safe to run anytime
- No changes are made to any branches

## Examples
- **List all branches**
  ```
  git-branch-list
  ```
  Shows all local and remote branches with summary

- **List only feature branches**
  ```
  git-branch-list "feature/"
  ```
  Shows only branches containing "feature/" in their name

- **Find branches with specific keyword**
  ```
  git-branch-list "auth"
  ```
  Shows all branches containing "auth" (e.g., feature/authentication)