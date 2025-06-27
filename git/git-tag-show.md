# Git Tag Show

## Purpose
Show detailed information about a tag

## Context
Use to inspect tag details including the tagged commit, tagger information, tag message, and associated changes. Helpful for understanding what a release contains or verifying tag information before deployment.

## Parameters
- `$ARGUMENTS` - Tag name to show
  - Optional: shows latest tag if not provided
  - Example: `v1.0.0`

## Steps

### 1. Determine tag to show
If no tag specified:
```bash
git describe --tags --abbrev=0
```
Finds the most recent tag.

### 2. Display tag information
```bash
git show "$TAG" --no-patch
```
Shows tag metadata without the full patch.

### 3. Show tagged commit
```bash
git show "$TAG" --oneline -s
```
Displays the commit the tag points to.

### 4. List changed files
```bash
git diff-tree --no-commit-id --name-status -r "$TAG"
```
Shows files modified in the tagged commit.

### 5. Show change statistics
```bash
git show "$TAG" --stat --no-patch
```
Displays summary statistics of changes.

## Validation
- Tag information is displayed
- Tagged commit is shown
- File changes are listed
- Statistics are accurate

## Error Handling
- **"fatal: ambiguous argument"** - Tag doesn't exist
- **"No tags found"** - Repository has no tags yet
- **"bad revision"** - Invalid tag name format

## Safety Notes
- This is a read-only operation
- Safe to run anytime
- No changes are made to repository

## Examples
- **Show latest tag**
  ```
  git-tag-show
  ```
  Displays information about the most recent tag

- **Show specific release**
  ```
  git-tag-show v2.1.0
  ```
  Shows details of version 2.1.0 tag

- **Inspect hotfix tag**
  ```
  git-tag-show hotfix-security-patch
  ```
  Reviews hotfix tag information