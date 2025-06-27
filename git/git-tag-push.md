Push tags to remote repository

```bash
# Parse arguments
TAG=$ARGUMENTS

if [ -z "$TAG" ]; then
    # Push all tags
    git push origin --tags
    echo "✓ Pushed all tags to origin"
    
    # Show pushed tags
    echo ""
    echo "Tags pushed:"
    git tag -l | tail -10
elif [ "$TAG" = "--follow-tags" ]; then
    # Push commits with their tags
    git push --follow-tags
    echo "✓ Pushed commits with associated tags"
else
    # Push specific tag
    git push origin "$TAG"
    echo "✓ Pushed tag: $TAG"
    
    # Show tag info
    echo ""
    git show "$TAG" --no-patch
fi
```