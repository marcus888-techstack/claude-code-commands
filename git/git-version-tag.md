Create semantic version tag with proper format

```bash
# Parse version and message from arguments
VERSION=$(echo $ARGUMENTS | cut -d' ' -f1)
MESSAGE=$(echo $ARGUMENTS | cut -d' ' -f2-)

# Validate version format
if [[ ! "$VERSION" =~ ^v[0-9]+\.[0-9]+\.[0-9]+(-[a-zA-Z0-9\.\-]+)?(\+[a-zA-Z0-9\.\-]+)?$ ]]; then
    echo "Error: Invalid version format. Use: v1.2.3[-prerelease][+build]"
    echo "Examples: v1.0.0, v2.1.0-beta.1, v1.0.0-rc.1+build.123"
    exit 1
fi

# Create annotated tag
if [ -z "$MESSAGE" ]; then
    git tag -a "$VERSION" -m "Release $VERSION"
else
    git tag -a "$VERSION" -m "$MESSAGE"
fi

# Push tag
git push origin "$VERSION"

echo "✓ Created tag: $VERSION"
git show "$VERSION" --no-patch
```