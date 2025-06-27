# Git Version Prerelease

## Purpose
Create pre-release tags (alpha/beta/rc)

## Context
Use to create pre-release versions for testing before the final release. Follows the standard progression: alpha → beta → release candidate (rc) → final release. Each stage represents increasing stability.

## Parameters
- `$ARGUMENTS` - Version, type, and optional number
  - Required: base version and pre-release type
  - Format: `version type [number]`
  - Example: `v2.0.0 beta 1` or `v2.0.0 alpha`

## Steps

### 1. Parse arguments
```bash
VERSION=$(echo $ARGUMENTS | cut -d' ' -f1)
TYPE=$(echo $ARGUMENTS | cut -d' ' -f2)
NUMBER=$(echo $ARGUMENTS | cut -d' ' -f3)
```
Extracts version, pre-release type, and optional number.

### 2. Validate inputs
- Version must match: `vX.Y.Z`
- Type must be: `alpha`, `beta`, or `rc`
- Number is optional for iteration

### 3. Build pre-release tag
- Without number: `v2.0.0-beta`
- With number: `v2.0.0-beta.1`

### 4. Create and push tag
```bash
git tag -a "$PRERELEASE_TAG" -m "Pre-release: $PRERELEASE_TAG"
git push origin "$PRERELEASE_TAG"
```

### 5. Display pre-release sequence
Shows the typical progression path for the version.

## Validation
- Version format is correct
- Pre-release type is valid
- Tag is created and pushed
- Tag follows semantic versioning

## Error Handling
- **"Invalid version format"** - Use vX.Y.Z format
- **"Invalid pre-release type"** - Must be alpha, beta, or rc
- **"tag already exists"** - Pre-release already created

## Safety Notes
- Alpha: Early testing, expect bugs
- Beta: Feature complete, fixing bugs
- RC: Release candidate, final testing
- Number versions for multiple iterations

## Examples
- **Create first alpha**
  ```
  git-version-prerelease v2.0.0 alpha
  ```
  Creates v2.0.0-alpha

- **Create numbered beta**
  ```
  git-version-prerelease v2.0.0 beta 2
  ```
  Creates v2.0.0-beta.2

- **Create release candidate**
  ```
  git-version-prerelease v2.0.0 rc 1
  ```
  Creates v2.0.0-rc.1