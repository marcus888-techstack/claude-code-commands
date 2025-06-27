Create an annotated tag and push to remote

```bash
# Create annotated tag
git tag -a $ARGUMENTS -m "Tag: $ARGUMENTS"

# Push tag to remote
git push origin $ARGUMENTS

# Show tag info
git show $ARGUMENTS
```