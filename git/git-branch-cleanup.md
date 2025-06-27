Clean up merged branches except protected ones

```bash
# Delete local merged branches except master, develop, and specified branch
git branch --merged | grep -v -E "master|develop|$ARGUMENTS" | xargs -n 1 git branch -d

# Prune remote tracking branches
git remote prune origin

# Show remaining branches
git branch -av
```