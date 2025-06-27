# Git Utility Log

## Purpose
View formatted git history with various options

## Context
Use to explore repository history in different ways. Provides pre-configured views for common scenarios like feature branches, releases, authors, and file changes. More convenient than remembering complex git log options.

## Parameters
- `$ARGUMENTS` - Log view type or custom options
  - Optional: defaults to graph view
  - Options: `feature`, `release`, `author`, `files`, `search`, `between`
  - Example: `feature` or custom git log options

## Steps

### 1. Default view (no arguments)
```bash
git log --oneline --graph --decorate --all -20
```
Shows graphical view of recent commits across all branches.

### 2. Feature branch history
```bash
git log --oneline --graph --first-parent develop
```
Shows feature merges into develop branch.

### 3. Release history
```bash
git log --oneline --decorate --tags --no-walk
```
Lists all release tags without intermediate commits.

### 4. Author statistics
```bash
git shortlog -sn --all
```
Shows commit count by author.

### 5. File changes view
```bash
git log --stat --oneline -10
```
Displays recent commits with file statistics.

### 6. Search commits
Prompts for search term and finds matching commits.

### 7. Between tags
Shows commits between two specified tags.

## Validation
- Output format is readable
- Options produce expected results
- Search finds relevant commits
- Tag ranges are valid

## Error Handling
- **"fatal: bad revision"** - Invalid tag or branch name
- **"No matches found"** - Search term not in history
- **Invalid option** - Falls back to standard git log

## Safety Notes
- This is a read-only operation
- Safe to run anytime
- Large histories may take time to display
- Use pagination for long outputs

## Examples
- **View branch graph**
  ```
  git-utility-log
  ```
  Shows visual branch structure

- **View feature history**
  ```
  git-utility-log feature
  ```
  Shows features merged into develop

- **Search for commits**
  ```
  git-utility-log search
  ```
  Prompts for search term

- **Custom format**
  ```
  git-utility-log --since="2 weeks ago" --author="John"
  ```
  Uses custom git log options