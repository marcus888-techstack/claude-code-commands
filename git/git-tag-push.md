# Git Tag Push

## Purpose
Push tags to remote repository

## Context
Use to share tags with your team through the remote repository. Supports pushing all tags, specific tags, or tags along with commits. Essential after creating local tags that need to be available for deployment or releases.

## Parameters
- `$ARGUMENTS` - Tag name or push option
  - Optional
  - Options: specific tag name, `--follow-tags`, or empty for all tags
  - Example: `v1.0.0` or `--follow-tags`

## Steps

### 1. Determine push mode
Based on arguments:
- No arguments: Push all tags
- `--follow-tags`: Push commits with associated tags
- Specific tag: Push only that tag

### 2A. Push all tags
```bash
git push origin --tags
```
Pushes all local tags to remote.

### 2B. Push with follow-tags
```bash
git push --follow-tags
```
Pushes commits and any associated annotated tags.

### 2C. Push specific tag
```bash
git push origin "$TAG"
```
Pushes only the specified tag.

### 3. Display confirmation
Shows what was pushed and tag information.

## Validation
- Tags appear on remote: `git ls-remote --tags origin`
- No error messages during push
- Specific tag (if specified) is on remote

## Error Handling
- **"tag does not exist"** - Check tag name with `git tag -l`
- **"failed to push"** - Check network and permissions
- **"already exists"** - Tag already on remote
- **"non-fast-forward"** - Tag exists with different commit

## Safety Notes
- Pushed tags are public and hard to remove
- Ensure tags are correct before pushing
- Coordinate with team for release tags
- Use annotated tags for releases

## Examples
- **Push all tags**
  ```
  git-tag-push
  ```
  Shares all local tags with remote

- **Push specific release**
  ```
  git-tag-push v2.0.0
  ```
  Pushes only the v2.0.0 tag

- **Push commits with tags**
  ```
  git-tag-push --follow-tags
  ```
  Pushes commits and their annotated tags