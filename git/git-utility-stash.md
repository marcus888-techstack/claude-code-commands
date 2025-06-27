# Git Utility Stash

## Purpose
Stash and manage work in progress changes

## Context
Use to temporarily save uncommitted changes when you need to switch branches or pull updates. Stashes act like a stack of temporary commits that can be reapplied later. Essential for managing interrupted work.

## Parameters
- `$ARGUMENTS` - Stash command and options
  - Optional: defaults to creating a stash
  - Commands: `save message`, `list`, `pop`, `apply [ref]`, `clear`
  - Example: `save "WIP: feature implementation"`

## Steps

### 1. Create stash (default)
```bash
git stash push -m "WIP: $(date +%Y-%m-%d-%H%M%S)"
```
Saves current changes with timestamp.

### 2. Save with message
```bash
git stash push -m "$MESSAGE"
```
Creates stash with descriptive message.

### 3. List stashes
```bash
git stash list
```
Shows all saved stashes with references.

### 4. Pop stash
```bash
git stash pop
```
Applies and removes the latest stash.

### 5. Apply stash
```bash
git stash apply [stash@{n}]
```
Applies stash without removing it.

### 6. Clear all stashes
```bash
git stash clear
```
Removes all stashes (destructive).

## Validation
- Changes are successfully stashed
- Working directory is clean after stash
- Stash can be reapplied
- No conflicts on pop/apply

## Error Handling
- **"No local changes"** - Nothing to stash
- **"CONFLICT"** - Stash conflicts with current changes
- **"stash@{n} is not a stash"** - Invalid stash reference
- **"Cannot apply to a dirty tree"** - Commit or stash current changes first

## Safety Notes
- Stashes are local only (not pushed)
- Clear command is destructive
- Conflicts can occur when applying old stashes
- Include untracked files with -u flag

## Examples
- **Quick stash**
  ```
  git-utility-stash
  ```
  Stashes with automatic timestamp

- **Stash with description**
  ```
  git-utility-stash save "implementing user auth"
  ```
  Creates named stash

- **Apply specific stash**
  ```
  git-utility-stash apply stash@{2}
  ```
  Applies third stash in list

- **View all stashes**
  ```
  git-utility-stash list
  ```
  Shows stash stack