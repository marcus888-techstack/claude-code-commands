# Git Version Tag

## Purpose
Create semantic version tag with proper format

## Context
Use to create version tags following semantic versioning standards. Ensures tags are properly formatted and includes support for pre-release and build metadata. Essential for proper release management.

## Parameters
- `$ARGUMENTS` - Version and optional message
  - Required: version (e.g., `v1.2.3`)
  - Optional: custom message after version
  - Example: `v1.2.3 "Major feature release"`

## Steps

### 1. Parse arguments
```bash
VERSION=$(echo $ARGUMENTS | cut -d' ' -f1)
MESSAGE=$(echo $ARGUMENTS | cut -d' ' -f2-)
```
Separates version from optional message.

### 2. Validate version format
Regex validation for semantic versioning:
```
^v[0-9]+\.[0-9]+\.[0-9]+(-[a-zA-Z0-9\.\-]+)?(\+[a-zA-Z0-9\.\-]+)?$
```
Supports:
- Standard: `v1.2.3`
- Pre-release: `v1.2.3-beta.1`
- Build metadata: `v1.2.3+build.123`

### 3. Create annotated tag
```bash
git tag -a "$VERSION" -m "$MESSAGE"
```
Creates tag with either custom or default message.

### 4. Push tag to remote
```bash
git push origin "$VERSION"
```
Shares the tag with the team.

### 5. Display tag information
Shows the created tag details for confirmation.

## Validation
- Version follows semantic versioning format
- Tag is created locally
- Tag is pushed to remote
- Tag information displays correctly

## Error Handling
- **"Invalid version format"** - Must follow semantic versioning
- **"tag already exists"** - Version already tagged
- **"failed to push"** - Check network and permissions

## Safety Notes
- Version tags should be immutable once pushed
- Follow semantic versioning rules:
  - MAJOR: Breaking changes
  - MINOR: New features (backward compatible)
  - PATCH: Bug fixes
- Use pre-release tags for testing

## Examples
- **Create standard release**
  ```
  git-version-tag v1.2.3
  ```
  Creates tag with default message

- **Create with custom message**
  ```
  git-version-tag v2.0.0 "Major API redesign"
  ```
  Creates tag with specific message

- **Create pre-release**
  ```
  git-version-tag v1.0.0-beta.1
  ```
  Creates beta version tag