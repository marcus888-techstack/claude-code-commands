# Git Tag Delete

## Purpose
Delete tags locally and remotely

## Context
Use when you need to remove a tag that was created by mistake or is no longer needed. This removes the tag from both your local repository and the remote. Be cautious as tags are often used for releases.

## Parameters
- `$ARGUMENTS` - Name of the tag to delete
  - Required
  - Example: `v1.0.0-beta` or `old-release`

## Steps

### 1. Validate tag input
Ensures tag name is provided.

### 2. Check and delete local tag
```bash
if git rev-parse "$TAG" >/dev/null 2>&1; then
    git tag -d "$TAG"
fi
```
Removes tag from local repository if it exists.

### 3. Check and delete remote tag
```bash
if git ls-remote --tags origin | grep -q "refs/tags/$TAG"; then
    git push origin --delete "$TAG"
fi
```
Removes tag from remote repository if it exists.

### 4. Confirm deletion
Shows confirmation that tag has been removed.

## Validation
- Local tag no longer in `git tag -l`
- Remote tag no longer in `git ls-remote --tags`
- No error messages for non-existent tags

## Error Handling
- **"Please specify tag to delete"** - Tag name required
- **"Local tag not found"** - Tag doesn't exist locally
- **"Remote tag not found"** - Tag doesn't exist on remote
- **Permission denied** - Check repository permissions

## Safety Notes
- Tag deletion is permanent
- Ensure tag is not being used for production
- Communicate with team before deleting release tags
- Consider if tag is referenced in documentation

## Examples
- **Delete a mistaken tag**
  ```
  git-tag-delete v1.0.0-wrong
  ```
  Removes incorrect tag from local and remote

- **Clean up old beta tag**
  ```
  git-tag-delete beta-old-feature
  ```
  Removes outdated beta tag