# Git Utility Reset

## Purpose
Reset repository to a specific state

## Context
Use to undo commits or move HEAD to a different state. Three reset modes offer different levels of preservation for your changes. Essential for correcting mistakes or reorganizing commit history.

## Parameters
- `$ARGUMENTS` - Reset type and target
  - Format: `[soft|mixed|hard] [commit|HEAD~n]`
  - Example: `soft HEAD~1` or `hard origin/main`

## Steps

### 1. Parse reset type
- **soft**: Keeps changes staged
- **mixed**: Keeps changes unstaged (default)
- **hard**: Discards all changes

### 2. Determine target
Default target is HEAD~1 if not specified.

### 3. Confirm destructive operations
For hard reset:
- Display warning about data loss
- Show target commit
- Require explicit confirmation

### 4. Execute reset
```bash
git reset --$TYPE $TARGET
```

### 5. Show resulting status
Displays current working directory state.

## Validation
- Reset completes successfully
- Working directory matches expected state
- Changes are preserved/discarded as specified
- HEAD points to correct commit

## Error Handling
- **"fatal: ambiguous argument"** - Invalid commit reference
- **"Cannot do hard reset with pending merge"** - Resolve merge first
- **"unknown option"** - Invalid reset type
- **No target specified** - Defaults to HEAD~1

## Safety Notes
- Hard reset is destructive and irreversible
- Soft/mixed preserve work in different states
- Cannot reset with uncommitted merge
- Consider stashing before hard reset
- Affects only local repository

## Examples
- **Undo last commit, keep changes staged**
  ```
  git-utility-reset soft HEAD~1
  ```
  Useful for amending commits

- **Reset to remote state**
  ```
  git-utility-reset hard origin/main
  ```
  Discards all local changes

- **Unstage all changes**
  ```
  git-utility-reset mixed HEAD
  ```
  Moves staged changes to working directory