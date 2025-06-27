# Git Utility Changelog

## Purpose
Generate changelog from git tags and commits

## Context
Use to automatically create a changelog from your git history. Can generate a full changelog from all version tags or a specific range. Supports grouping by commit type (features, fixes) when using conventional commits.

## Parameters
- `$ARGUMENTS` - Version range or empty for full changelog
  - Optional
  - Example: `v1.0.0..v2.0.0` or `HEAD~10..HEAD`

## Steps

### 1. Full changelog (no arguments)
For each version tag (newest first):
- Display version and release date
- List commits since previous version
- Exclude merge commits
- Format as markdown

### 2. Range changelog (with arguments)
For specified range:
- Group commits by type (feat, fix, other)
- Display in categorized sections
- Use conventional commit format

### 3. Format output
```bash
# Full changelog
git log $TAG...$PREV_TAG --pretty=format:"- %s" --no-merges

# Grouped by type
git log $RANGE --grep="^feat" --pretty=format:"- %s" --no-merges
```

## Validation
- Tags are found and sorted correctly
- Commit messages are formatted properly
- Date information is accurate
- Markdown formatting is valid

## Error Handling
- **No tags found** - Repository has no version tags
- **Invalid range** - Check range syntax (TAG1..TAG2)
- **Empty changelog** - No commits in specified range

## Safety Notes
- This is a read-only operation
- Output can be piped to CHANGELOG.md
- Consider commit message quality for best results
- Use conventional commits for better grouping

## Examples
- **Generate full changelog**
  ```
  git-utility-changelog
  ```
  Creates changelog from all version tags

- **Generate changelog for range**
  ```
  git-utility-changelog v1.0.0..v2.0.0
  ```
  Shows changes between two versions

- **Recent changes**
  ```
  git-utility-changelog HEAD~20..HEAD
  ```
  Shows last 20 commits grouped by type