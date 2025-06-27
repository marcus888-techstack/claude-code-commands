Delete tags locally and remotely

```bash
# Tag to delete
TAG=$ARGUMENTS

if [ -z "$TAG" ]; then
    echo "Error: Please specify tag to delete"
    echo "Usage: /git/tag/delete tag-name"
    exit 1
fi

# Check if tag exists locally
if git rev-parse "$TAG" >/dev/null 2>&1; then
    # Delete local tag
    git tag -d "$TAG"
    echo "✓ Deleted local tag: $TAG"
else
    echo "⚠️  Local tag not found: $TAG"
fi

# Check if tag exists remotely
if git ls-remote --tags origin | grep -q "refs/tags/$TAG"; then
    # Delete remote tag
    git push origin --delete "$TAG"
    echo "✓ Deleted remote tag: $TAG"
else
    echo "⚠️  Remote tag not found: $TAG"
fi

echo ""
echo "Tag '$TAG' has been removed"
```