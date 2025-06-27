Rename a branch locally and update remote

```bash
# Parse arguments (expects: "old-name new-name")
OLD_NAME=$(echo $ARGUMENTS | cut -d' ' -f1)
NEW_NAME=$(echo $ARGUMENTS | cut -d' ' -f2)

# Rename local branch
git branch -m $OLD_NAME $NEW_NAME

# Delete old remote branch and push new one
git push origin --delete $OLD_NAME
git push origin -u $NEW_NAME
```