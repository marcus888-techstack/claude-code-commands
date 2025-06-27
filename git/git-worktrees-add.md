# Git Worktrees Add

## Purpose
Add a new Git worktree

## Context
Use to work on multiple branches simultaneously without switching. Worktrees create separate working directories for different branches, allowing parallel development. Ideal for bug fixes while working on features, or comparing implementations.

## Parameters
- `$ARGUMENTS` - Path and branch specification
  - Format: `<path> <branch>` or `<path> -b <new-branch>`
  - Example: `../features/oauth feature/oauth`

## Steps

### 1. Parse arguments
Extracts path and branch information from arguments.

### 2A. Create worktree with new branch
```bash
git worktree add -b "$BRANCH_NAME" "$PATH_ARG"
```
Creates new branch and worktree simultaneously.

### 2B. Create worktree for existing branch
```bash
git worktree add "$PATH_ARG" "$BRANCH_NAME"
```
Links existing branch to new worktree.

### 3. Display worktree information
Shows:
- Full path to worktree
- Associated branch name
- Active status

### 4. Provide next steps
Guides user on how to use the new worktree.

## Validation
- Worktree directory is created
- Branch is properly linked
- Can navigate to worktree
- Git commands work in new location

## Error Handling
- **"branch already exists"** - Branch name is taken
- **"already exists"** - Path is already used
- **"not a valid directory"** - Parent directory missing
- **"branch is already checked out"** - Branch in use elsewhere

## Safety Notes
- Each branch can only have one worktree
- Worktrees share the same repository
- Changes are immediately visible across worktrees
- Clean up worktrees when done to save space

## Examples
- **Add worktree for existing branch**
  ```
  git-worktrees-add ../features/oauth feature/oauth
  ```
  Creates worktree for oauth feature

- **Create new branch with worktree**
  ```
  git-worktrees-add ../features/payment -b feature/payment
  ```
  Creates both branch and worktree

- **Hotfix while developing**
  ```
  git-worktrees-add ../hotfix/urgent hotfix/security-fix
  ```
  Work on hotfix without interrupting feature