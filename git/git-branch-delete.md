Delete a branch locally and remotely

```bash
# Delete local branch
git branch -d $ARGUMENTS

# Delete remote branch
git push origin --delete $ARGUMENTS

# Force delete if branch has unmerged changes
git branch -D $ARGUMENTS
```