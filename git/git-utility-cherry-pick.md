Cherry-pick specific commits to current branch

```bash
# Cherry-pick single commit
git cherry-pick $ARGUMENTS

# If conflicts occur, resolve and continue
git add .
git cherry-pick --continue
```