# Git Version Dev Tag

## Purpose
Tag development version with _dev suffix

## Context
Use to mark development versions on the develop branch. The _dev suffix distinguishes these from production releases and helps track what's currently in development.

## Parameters
- `$ARGUMENTS` - Version to tag
  - Optional: uses latest tag if not provided
  - Example: `v1.2.3` or `1.2.3`

## Steps

### 1. Determine version
```bash
if [ -z "$ARGUMENTS" ]; then
    VERSION=$(git describe --tags --abbrev=0 | sed 's/_dev$//')
else
    VERSION=$ARGUMENTS
fi
```
Uses provided version or finds the latest tag.

### 2. Ensure version format
```bash
if [[ ! "$VERSION" =~ ^v ]]; then
    VERSION="v$VERSION"
fi
```
Adds 'v' prefix if missing.

### 3. Create development tag
```bash
DEV_VERSION="${VERSION}_dev"
git tag -a "$DEV_VERSION" -m "Development version $DEV_VERSION"
```
Appends _dev suffix to mark as development.

### 4. Push tag to remote
```bash
git push origin "$DEV_VERSION"
```
Shares the development tag with the team.

## Validation
- Version format is correct
- _dev suffix is added
- Tag is created locally
- Tag is pushed to remote

## Error Handling
- **"No names found"** - No existing tags to base on
- **"tag already exists"** - Development tag already created
- **"failed to push"** - Check network and permissions

## Safety Notes
- Development tags indicate work in progress
- Should only be used on develop branch
- Not for production deployment
- Update after merging features

## Examples
- **Tag current development version**
  ```
  git-version-dev-tag
  ```
  Uses latest tag and adds _dev

- **Tag specific version**
  ```
  git-version-dev-tag v1.3.0
  ```
  Creates v1.3.0_dev tag

- **After feature merge**
  ```
  git-version-dev-tag 1.2.4
  ```
  Marks new development version