# Git Tag Create

## Purpose
Create an annotated tag and push to remote

## Context
Use to mark specific points in your repository's history. Annotated tags are recommended for releases as they contain tagger information and can be signed. Tags are commonly used for version releases and important milestones.

## Parameters
- `$ARGUMENTS` - Tag name and optionally a custom message
  - Required
  - Example: `v1.0.0` or `v1.0.0 "First stable release"`

## Steps

### 1. Create annotated tag
```bash
git tag -a $ARGUMENTS -m "Tag: $ARGUMENTS"
```
Creates an annotated tag with metadata.

### 2. Push tag to remote
```bash
git push origin $ARGUMENTS
```
Shares the tag with the remote repository.

### 3. Show tag information
```bash
git show $ARGUMENTS
```
Displays the tag details and associated commit.

## Validation
- Tag is created locally
- Tag appears in `git tag -l`
- Tag is pushed to remote
- Tag information shows correctly

## Error Handling
- **"fatal: tag already exists"** - Tag name is taken, use different name
- **"error: failed to push"** - Check network and permissions
- **Invalid tag name** - Use valid characters (no spaces, special chars)

## Safety Notes
- Tags should be treated as permanent
- Use semantic versioning for releases (v1.0.0)
- Consider signing tags for security (-s flag)
- Document what the tag represents

## Examples
- **Create version tag**
  ```
  git-tag-create v1.0.0
  ```
  Creates and pushes version 1.0.0 tag

- **Create milestone tag**
  ```
  git-tag-create beta-release
  ```
  Tags current commit as beta-release