Tag development version with _dev suffix

```bash
# Get version from arguments or current branch
if [ -z "$ARGUMENTS" ]; then
    # Try to get from latest tag
    VERSION=$(git describe --tags --abbrev=0 | sed 's/_dev$//')
else
    VERSION=$ARGUMENTS
fi

# Ensure version format
if [[ ! "$VERSION" =~ ^v ]]; then
    VERSION="v$VERSION"
fi

# Create dev tag
DEV_VERSION="${VERSION}_dev"
git tag -a "$DEV_VERSION" -m "Development version $DEV_VERSION"
git push origin "$DEV_VERSION"

echo "✓ Created development tag: $DEV_VERSION"
```