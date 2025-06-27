Sync branch with remote upstream

```bash
# Fetch latest changes
git fetch origin

# Sync specified branch
git checkout $ARGUMENTS
git pull origin $ARGUMENTS

# Show status
git status
git log --oneline -5
```