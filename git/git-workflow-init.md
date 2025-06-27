# Git Workflow Init

## Purpose
Initialize Git workflow with main and develop branches

## Context
Use to set up a repository with the git-flow branching model. Creates the necessary branches, sets protection, and establishes initial versioning. This simplified workflow uses tags instead of release branches.

## Parameters
- None required

## Steps

### 1. Ensure on main branch
```bash
git checkout main
git pull origin main
```
Starts from the production branch.

### 2. Create develop branch (if needed)
```bash
if ! git show-ref --verify --quiet refs/heads/develop; then
    git checkout -b develop
    git push -u origin develop
fi
```
Creates the integration branch for features.

### 3. Set branch protection
```bash
git config --global branch.main.protect true
git config --global branch.develop.protect true
```
Marks branches as protected in local config.

### 4. Create initial version tag (if needed)
```bash
if [ -z "$(git tag -l)" ]; then
    git tag -a "v0.1.0" -m "Initial release"
    git push origin v0.1.0
    
    git checkout develop
    git tag -a "v0.1.0_dev" -m "Development starts from v0.1.0"
    git push origin v0.1.0_dev
fi
```
Establishes initial versioning for both branches.

### 5. Display workflow summary
Shows the initialized structure and workflow guidelines.

## Validation
- Main and develop branches exist
- Both branches have protection config
- Initial tags are created
- Remote tracking is set up

## Error Handling
- **Branch already exists** - Skips creation, continues setup
- **No remote 'origin'** - Ensure remote is configured
- **Permission denied** - Check repository permissions

## Safety Notes
- Run only once per repository
- Ensure you have push permissions
- Team should agree on workflow before initialization
- Consider existing branches before running

## Examples
- **Initialize new repository**
  ```
  git-workflow-init
  ```
  Sets up complete git-flow structure

- **After cloning repository**
  ```
  git-workflow-init
  ```
  Ensures all workflow branches exist locally