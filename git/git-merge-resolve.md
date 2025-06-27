# Git Merge Resolve

## Purpose
Help resolve merge conflicts step by step

## Context
Use when you encounter merge conflicts and need guidance on resolving them. This command identifies conflicted files and provides options for resolution, making the conflict resolution process more manageable.

## Parameters
- `$ARGUMENTS` - Specific file to check for conflicts
  - Optional
  - Example: `src/main.js`

## Steps

### 1. Check for merge conflicts
```bash
git diff --name-only --diff-filter=U
```
Lists all files with unresolved conflicts.

### 2. Handle no conflicts case
If no conflicts found, exit with success message.

### 3. Display conflict information
- List all conflicted files with numbers
- Show resolution options available
- Provide specific guidance if file is specified

### 4. For specific file (if provided)
```bash
grep -n "^<<<<<<< \|^======= \|^>>>>>>> " "$FILE"
```
Shows conflict markers and their line numbers in the specified file.

### 5. Provide resolution instructions
- Manual editing steps
- How to mark as resolved
- How to complete the merge

## Validation
- All conflicts are identified
- Clear instructions are provided
- Conflict markers are visible for specific files
- User knows next steps

## Error Handling
- **"No merge conflicts detected"** - Good news, nothing to resolve
- **"No conflict markers found"** - File may already be resolved
- **File not found** - Check file name and path

## Safety Notes
- Always review resolved conflicts before committing
- Test code after resolving conflicts
- Understand both sides of the conflict before choosing
- Keep backup of complex resolutions

## Examples
- **Check all conflicts**
  ```
  git-merge-resolve
  ```
  Lists all conflicted files with resolution options

- **Check specific file**
  ```
  git-merge-resolve src/config.js
  ```
  Shows conflict details for src/config.js