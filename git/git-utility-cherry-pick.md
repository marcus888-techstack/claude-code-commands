# Git Utility Cherry-pick

## Purpose
Cherry-pick specific commits to current branch

## Context
Use to apply specific commits from another branch without merging the entire branch. Useful for backporting fixes, applying specific features, or recovering lost commits. Maintains original commit authorship.

## Parameters
- `$ARGUMENTS` - Commit hash or range to cherry-pick
  - Required
  - Example: `abc123` or `abc123..def456`

## Steps

### 1. Apply the commit(s)
```bash
git cherry-pick $ARGUMENTS
```
Applies the specified commit(s) to current branch.

### 2. Handle conflicts (if any)
If conflicts occur:
```bash
# Resolve conflicts in files
git add .
git cherry-pick --continue
```

### 3. Alternative actions
- Abort: `git cherry-pick --abort`
- Skip: `git cherry-pick --skip`

## Validation
- Commit is successfully applied
- Original commit message is preserved
- Authorship information is maintained
- No unresolved conflicts remain

## Error Handling
- **"bad revision"** - Commit hash doesn't exist
- **"CONFLICT"** - Merge conflicts need resolution
- **"empty commit"** - Commit makes no changes
- **"cherry-pick is already in progress"** - Previous pick not completed

## Safety Notes
- Cherry-picking changes commit hashes
- Can create duplicate commits if not careful
- Preserve commit order when picking multiple
- Consider merge instead for multiple commits

## Examples
- **Cherry-pick single commit**
  ```
  git-utility-cherry-pick abc123def
  ```
  Applies commit abc123def to current branch

- **Cherry-pick range**
  ```
  git-utility-cherry-pick feature~3..feature
  ```
  Applies last 3 commits from feature branch

- **Cherry-pick merge commit**
  ```
  git-utility-cherry-pick -m 1 merge123
  ```
  Picks a merge commit specifying parent