# Git Version Increment

## Purpose
Increment version based on change type (major/minor/patch)

## Context
Use to calculate the next version number based on semantic versioning rules. This shows what the next version should be without actually creating it, helping you plan releases according to the type of changes made.

## Parameters
- `$ARGUMENTS` - Type of increment
  - Required
  - Options: `major`, `minor`, or `patch`
  - Example: `minor`

## Steps

### 1. Get current version
```bash
git describe --tags --abbrev=0 | sed 's/v//' | sed 's/_dev$//'
```
Finds the latest tag and strips version prefix/suffix.

### 2. Parse version components
```bash
MAJOR=$(echo $CURRENT_VERSION | cut -d. -f1)
MINOR=$(echo $CURRENT_VERSION | cut -d. -f2)
PATCH=$(echo $CURRENT_VERSION | cut -d. -f3 | cut -d- -f1)
```
Extracts major, minor, and patch numbers.

### 3. Calculate new version
Based on increment type:
- **major**: Increment major, reset minor and patch to 0
- **minor**: Increment minor, reset patch to 0
- **patch**: Increment patch only

### 4. Display version change
Shows current version → new version with change type context.

### 5. Suggest next steps
Provides commands for creating release branch or tag.

## Validation
- Current version exists and is parsed correctly
- New version follows semantic versioning
- Increment type is valid

## Error Handling
- **"Usage: /git/version/increment"** - Invalid increment type
- **"fatal: No names found"** - No version tags exist yet
- **Invalid version format** - Current tag doesn't follow semver

## Safety Notes
- This only calculates, doesn't create the version
- Ensure changes match the increment type:
  - major: Breaking changes
  - minor: New features (backward compatible)
  - patch: Bug fixes only

## Examples
- **Calculate major version bump**
  ```
  git-version-increment major
  ```
  Shows: 1.2.3 → 2.0.0 (Breaking change)

- **Calculate minor version bump**
  ```
  git-version-increment minor
  ```
  Shows: 1.2.3 → 1.3.0 (New feature)

- **Calculate patch version bump**
  ```
  git-version-increment patch
  ```
  Shows: 1.2.3 → 1.2.4 (Bug fix)