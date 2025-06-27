Show detailed information about a tag

```bash
# Tag to show
TAG=$ARGUMENTS

if [ -z "$TAG" ]; then
    # Show latest tag
    TAG=$(git describe --tags --abbrev=0)
    echo "Showing latest tag: $TAG"
    echo ""
fi

# Show tag details
echo "=== Tag Information ==="
git show "$TAG" --no-patch

echo ""
echo "=== Tag Commit ==="
git show "$TAG" --oneline -s

echo ""
echo "=== Files Changed ==="
git diff-tree --no-commit-id --name-status -r "$TAG"

echo ""
echo "=== Tag Statistics ==="
git show "$TAG" --stat --no-patch
```