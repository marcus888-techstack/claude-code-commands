# Git Merge Squash

## Purpose
Squash merge commits into a single commit

## Context
Use when you want to merge a feature branch but condense all its commits into one clean commit. This keeps the main branch history linear and removes noisy commit messages from development. Ideal for features with many small commits.

## Parameters
- `$ARGUMENTS` - Name of the branch to squash merge
  - Required
  - Example: `feature/messy-development`

## Steps

### 1. Validate input
Check that a branch name is provided.

### 2. Get current branch
```bash
git branch --show-current
```
Identifies which branch you're merging into.

### 3. Perform squash merge
```bash
git merge --squash $BRANCH
```
Stages all changes from the branch without creating a merge commit.

### 4. Display staged changes
```bash
git status --short
```
Shows all files that will be included in the squashed commit.

### 5. Provide commit instructions
Instructs user to create a single commit with all changes.

## Validation
- All branch changes are staged
- No merge commit is created yet
- Working directory shows staged files
- User can review before committing

## Error Handling
- **"Please specify branch to merge"** - Branch name is required
- **"merge: {branch} - not something we can merge"** - Branch doesn't exist
- **"CONFLICT (content)"** - Resolve conflicts before squashing
- **"Already up to date"** - No changes to merge from branch

## Safety Notes
- Review all staged changes before committing
- Write a comprehensive commit message
- Original branch history is not preserved
- Cannot easily revert individual commits after squashing

## Examples
- **Squash a feature with many commits**
  ```
  git-merge-squash feature/experimental-ui
  ```
  Stages all UI changes for a single commit

- **Clean up development history**
  ```
  git-merge-squash feature/refactor-auth
  ```
  Condenses refactoring work into one commit