# Git Version Bump

## Purpose
Bump version number in project files

## Context
Use to update version numbers in your project files (package.json, Cargo.toml, etc.) before creating a release. Supports semantic versioning with major, minor, and patch increments, or setting a specific version.

## Parameters
- `$ARGUMENTS` - Version increment type or specific version
  - Required
  - Options: `major`, `minor`, `patch`, or specific version like `1.2.3`
  - Example: `patch` or `2.0.0`

## Steps

### 1. Validate input
Check that version argument is provided.

### 2. Detect project type

#### For Node.js projects (package.json)
```bash
npm version $VERSION_ARG --no-git-tag-version
```
- Updates package.json and package-lock.json
- Stages changes
- Commits with standardized message
- Pushes to current branch

#### For Rust projects (Cargo.toml)
```bash
cargo set-version $VERSION_ARG
```
- Updates Cargo.toml and Cargo.lock
- Commits and pushes changes

#### For Python projects
- Prompts for manual update
- Provides commit message template

### 3. Commit version change
```bash
git commit -m "chore: bump version to v$NEW_VERSION"
git push origin $(git branch --show-current)
```

### 4. Display next steps
Suggests creating a release with the new version.

## Validation
- Version is updated in project files
- Changes are committed and pushed
- New version number is displayed

## Error Handling
- **"Usage: /git:version:bump"** - Version argument required
- **"No recognized project file"** - Manual update needed
- **npm version errors** - Check npm/node installation
- **Permission denied** - Check file permissions

## Safety Notes
- Ensure tests pass before bumping version
- Review changes before creating release
- Follow semantic versioning conventions
- Update changelog if applicable

## Examples
- **Bump patch version**
  ```
  git-version-bump patch
  ```
  Changes 1.0.0 to 1.0.1

- **Bump minor version**
  ```
  git-version-bump minor
  ```
  Changes 1.0.0 to 1.1.0

- **Set specific version**
  ```
  git-version-bump 2.0.0
  ```
  Sets version to 2.0.0