Get the latest version tag for a branch

```bash
# Get branch from arguments or use current
if [ -z "$ARGUMENTS" ]; then
    BRANCH=$(git branch --show-current)
else
    BRANCH=$ARGUMENTS
fi

echo "=== Latest Version Information ==="
echo ""
echo "Branch: $BRANCH"
echo ""

# Get latest tag
LATEST_TAG=$(git describe --tags --abbrev=0 $BRANCH 2>/dev/null)

if [ -z "$LATEST_TAG" ]; then
    echo "No tags found on branch $BRANCH"
    exit 1
fi

echo "Latest tag: $LATEST_TAG"
echo ""

# Show tag details
git show $LATEST_TAG --no-patch

# Show commits since tag
COMMITS_SINCE=$(git rev-list $LATEST_TAG..$BRANCH --count)
echo ""
echo "Commits since $LATEST_TAG: $COMMITS_SINCE"

if [ $COMMITS_SINCE -gt 0 ]; then
    echo ""
    echo "Recent changes:"
    git log $LATEST_TAG..$BRANCH --oneline | head -5
fi
```