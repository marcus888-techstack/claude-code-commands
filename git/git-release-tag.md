# Git Release Tag

## Purpose
Create a release using tags (modern workflow without release branches)

## Context
Use to create a new release version. This modern workflow uses tags instead of release branches, simplifying the release process. Can be run from either develop (pre-release) or main (post-release) branch.

## Parameters
- `$ARGUMENTS` - Version number for the release
  - Required
  - Example: `v1.2.0` or `1.2.0` (v prefix added automatically)

## Steps

### 1. Validate version input
Ensures version is provided and adds 'v' prefix if missing.

### 2. Check current branch
Determines workflow based on whether you're on develop or main.

### 3A. Release from develop branch
- Pull latest develop changes
- Run tests (customizable)
- Create release tag: `git tag -a "$VERSION" -m "Release $VERSION"`
- Push tag to origin
- Merge to main with --no-ff
- Push main branch
- Tag develop as `${VERSION}_dev` for continued development

### 3B. Release from main branch
- Pull latest main changes
- Create release tag on main
- Push tag to origin
- Sync develop with main
- Tag develop as `${VERSION}_dev`

### 4. Complete release
Shows summary and next steps for deployment.

## Validation
- Release tag is created and pushed
- Main branch has the release
- Develop is synchronized
- Development version tag exists

## Error Handling
- **"Please specify version"** - Version number required
- **"Must be on 'develop' or 'main' branch"** - Switch to correct branch
- **Test failures** - Fix issues before releasing
- **Tag already exists** - Use different version number

## Safety Notes
- Run tests before creating release
- Ensure CI/CD passes
- Review changes in the release
- Plan deployment immediately after tagging
- Keep release notes updated

## Examples
- **Release from develop**
  ```
  git checkout develop
  git-release-tag v1.2.0
  ```
  Creates release and merges to main

- **Hotfix release from main**
  ```
  git checkout main
  git-release-tag v1.2.1
  ```
  Tags main and syncs with develop