# Git Worktrees Feature

## Purpose
Create a feature branch worktree

## Context
Use to start a new feature development in a separate worktree. Creates a feature branch from develop and sets up a dedicated workspace. Includes feature tracking file and ensures you're working from the latest develop branch.

## Parameters
- `$ARGUMENTS` - Feature name
  - Required
  - Example: `user-auth`, `payment-integration`, `issue-123-fix-login`

## Steps

### 1. Validate feature name
Ensures a feature name is provided.

### 2. Update develop branch
```bash
git checkout develop
git pull origin develop
```
Ensures feature starts from latest code.

### 3. Create branch name
Formats as `feature/<feature-name>`.

### 4A. Use existing branch
```bash
git worktree add "$WORKTREE_PATH" "$BRANCH_NAME"
```
If branch exists, creates worktree from it.

### 4B. Create new branch
```bash
git worktree add -b "$BRANCH_NAME" "$WORKTREE_PATH"
```
Creates new feature branch with worktree.

### 5. Set up feature workspace
Creates `.feature-info` file with:
- Feature name and branch
- Creation date and author
- Task checklist template

### 6. Display next steps
Guides user on how to proceed with development.

## Validation
- Feature name is provided
- Develop branch is up to date
- Worktree is created successfully
- Feature branch is properly linked
- Feature info file is created

## Error Handling
- **"Usage: /git/worktrees/feature"** - Missing feature name
- **"Branch already exists"** - Uses existing branch
- **"failed to create worktree"** - Path or permission issues
- **"failed to pull"** - Network or authentication issues

## Safety Notes
- Always starts from latest develop
- Preserves existing branches
- Creates organized directory structure
- Feature info helps track progress
- Separate workspace prevents conflicts

## Examples
- **Start new authentication feature**
  ```
  git-worktrees-feature user-auth
  ```
  Creates feature/user-auth branch and worktree

- **Work on issue fix**
  ```
  git-worktrees-feature issue-123-fix-login
  ```
  Creates descriptive feature branch

- **Continue existing feature**
  ```
  git-worktrees-feature payment-integration
  ```
  Creates worktree for existing feature branch