# Git Version List

## Purpose
List version tags in semantic order

## Context
Use to see all version tags in your repository, organized by type (production, development, pre-release). Tags are sorted in semantic version order, making it easy to track release history.

## Parameters
- `$ARGUMENTS` - Optional filter pattern
  - Optional
  - Example: `beta` or `v2.`

## Steps

### 1. Display version categories (no filter)

#### Production versions
```bash
git tag -l --sort=-version:refname | grep -E "^v[0-9]+\.[0-9]+\.[0-9]+$"
```
Shows stable release versions (e.g., v1.0.0).

#### Development versions
```bash
git tag -l --sort=-version:refname | grep "_dev$"
```
Shows development tags (e.g., v1.0.0_dev).

#### Pre-release versions
```bash
git tag -l --sort=-version:refname | grep -E "^v[0-9]+\.[0-9]+\.[0-9]+-(alpha|beta|rc)"
```
Shows alpha, beta, and release candidate versions.

### 2. Display filtered versions (with filter)
```bash
git tag -l --sort=-version:refname | grep "$FILTER"
```
Shows only tags matching the filter pattern.

### 3. Show total count
```bash
git tag -l | wc -l
```
Displays total number of tags in repository.

## Validation
- Tags are sorted in proper semantic version order
- Categories are correctly filtered
- Count matches actual number of tags

## Error Handling
- **No tags found** - Repository has no version tags yet
- **Empty results** - No tags match the filter
- **Invalid regex** - Check filter pattern syntax

## Safety Notes
- This is a read-only operation
- Safe to run anytime
- No changes are made to repository

## Examples
- **List all version tags**
  ```
  git-version-list
  ```
  Shows production, development, and pre-release versions

- **List beta versions only**
  ```
  git-version-list beta
  ```
  Shows only beta release tags

- **List v2.x versions**
  ```
  git-version-list "v2\."
  ```
  Shows all version 2 tags