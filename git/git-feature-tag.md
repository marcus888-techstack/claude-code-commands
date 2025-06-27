# Git Feature Tag

## Purpose
Tag feature merge in develop branch with incremented patch version

## Context
Use after merging a feature to develop to create a development version tag. This helps track which features are included in which development versions. Automatically increments the patch version and adds _dev suffix.

## Parameters
- `$ARGUMENTS` - Description of the feature for the tag message
  - Required
  - Example: `Added user authentication system`

## Steps

### 1. Get current develop version
```bash
git describe --tags --abbrev=0 develop | sed 's/_dev$//'
```
Finds the most recent tag on develop and removes the _dev suffix.

### 2. Parse version components
```bash
MAJOR=$(echo $CURRENT_VERSION | cut -d. -f1 | sed 's/v//')
MINOR=$(echo $CURRENT_VERSION | cut -d. -f2)
PATCH=$(echo $CURRENT_VERSION | cut -d. -f3)
```
Extracts major, minor, and patch numbers from the version.

### 3. Increment patch version
```bash
NEW_PATCH=$((PATCH + 1))
NEW_VERSION="v${MAJOR}.${MINOR}.${NEW_PATCH}_dev"
```
Increments patch number and adds _dev suffix for development version.

### 4. Create and push tag
```bash
git tag -a "$NEW_VERSION" -m "Feature: $ARGUMENTS"
git push origin "$NEW_VERSION"
```
Creates annotated tag with feature description and pushes to remote.

### 5. Display confirmation
Shows the new version tag and feature description.

## Validation
- New tag exists locally: `git tag -l | grep $NEW_VERSION`
- Tag is pushed to remote: `git ls-remote --tags origin`
- Tag message contains feature description

## Error Handling
- **"fatal: No names found"** - No existing tags, start with v0.0.1_dev
- **"error: tag already exists"** - Version already tagged, check for duplicates
- **"error: failed to push"** - Check network connection and permissions

## Safety Notes
- Ensure you're on develop branch before tagging
- Only tag after successful merge and tests
- Use descriptive messages for tag history
- Tags are permanent, choose version carefully

## Examples
- **Tag authentication feature**
  ```
  git-feature-tag "Added user authentication with JWT"
  ```
  Creates tag like v1.2.3_dev with description

- **Tag payment feature**
  ```
  git-feature-tag "Integrated Stripe payment processing"
  ```
  Tags the payment feature in develop