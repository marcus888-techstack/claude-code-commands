# Git Version Latest

## Purpose
Get the latest version tag for a branch

## Context
Use to find the most recent version tag on a branch and see what changes have been made since then. Helpful for understanding the current release status and what's pending for the next release.

## Parameters
- `$ARGUMENTS` - Branch name to check
  - Optional: uses current branch if not provided
  - Example: `develop` or `main`

## Steps

### 1. Determine branch
```bash
if [ -z "$ARGUMENTS" ]; then
    BRANCH=$(git branch --show-current)
else
    BRANCH=$ARGUMENTS
fi
```
Uses specified branch or current branch.

### 2. Find latest tag
```bash
git describe --tags --abbrev=0 $BRANCH
```
Gets the most recent tag reachable from the branch.

### 3. Display tag information
```bash
git show $LATEST_TAG --no-patch
```
Shows tag details including tagger and message.

### 4. Count commits since tag
```bash
git rev-list $LATEST_TAG..$BRANCH --count
```
Calculates how many commits have been made since the tag.

### 5. Show recent changes
If commits exist since tag:
```bash
git log $LATEST_TAG..$BRANCH --oneline | head -5
```
Lists the most recent changes not yet released.

## Validation
- Branch exists and is valid
- Tag is found on the branch
- Commit count is accurate
- Recent changes are displayed

## Error Handling
- **"No tags found on branch"** - Branch has no version tags yet
- **"fatal: ambiguous argument"** - Branch doesn't exist
- **"fatal: No names found"** - No tags in repository

## Safety Notes
- This is a read-only operation
- Safe to run anytime
- Helps prevent accidental releases

## Examples
- **Check current branch version**
  ```
  git-version-latest
  ```
  Shows latest tag on current branch

- **Check main branch version**
  ```
  git-version-latest main
  ```
  Shows latest production version

- **Check develop branch progress**
  ```
  git-version-latest develop
  ```
  Shows development version and pending changes