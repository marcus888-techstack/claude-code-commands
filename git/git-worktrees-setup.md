# Git Worktrees Setup

## Purpose
Initialize Git worktree structure for parallel development

## Context
Use to set up an organized worktree structure for efficient parallel development. Creates dedicated directories for main, develop, features, and hotfixes. Includes a convenience script for quick switching between worktrees.

## Parameters
- `$ARGUMENTS` - Optional project name
  - Optional: defaults to repository name
  - Example: `my-project`

## Steps

### 1. Determine project name
Uses provided name or extracts from repository.

### 2. Get repository location
```bash
git rev-parse --show-toplevel
```
Identifies current repository root.

### 3. Create directory structure
```bash
mkdir -p "$PROJECT_NAME"/{main,develop,features,hotfixes}
```
Sets up organized worktree directories.

### 4. Move repository to main
Relocates repository to main worktree if needed.

### 5. Create develop worktree
```bash
git worktree add ../develop develop
```
Sets up development branch worktree.

### 6. Create switch script
Generates `switch-worktree.sh` for easy navigation between worktrees.

### 7. Display structure
Shows created directory layout and usage instructions.

## Validation
- Git repository exists
- Directory structure is created
- Main repository is relocated
- Develop worktree is added
- Switch script is executable

## Error Handling
- **"not a git repository"** - Must run in git repo
- **"worktree already exists"** - Skips existing worktrees
- **Permission errors** - Need write access to parent directory
- **"cannot move repository"** - Repository may be in use

## Safety Notes
- Moves existing repository (non-destructive)
- Preserves all git history and settings
- Creates organized structure
- Switch script helps prevent confusion
- All worktrees share same repository

## Examples
- **Setup with default name**
  ```
  git-worktrees-setup
  ```
  Uses repository name for structure

- **Setup with custom name**
  ```
  git-worktrees-setup my-awesome-project
  ```
  Creates custom project structure

- **After setup usage**
  ```
  cd ../develop
  git worktree add ../features/new-feature -b feature/new-feature
  ```
  Creates feature worktree in organized structure