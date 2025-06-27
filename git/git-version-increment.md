Increment version based on change type (major/minor/patch)

```bash
# Get increment type from arguments
INCREMENT_TYPE=$ARGUMENTS

# Get current version
CURRENT_VERSION=$(git describe --tags --abbrev=0 | sed 's/v//' | sed 's/_dev$//')

# Parse version components
MAJOR=$(echo $CURRENT_VERSION | cut -d. -f1)
MINOR=$(echo $CURRENT_VERSION | cut -d. -f2)
PATCH=$(echo $CURRENT_VERSION | cut -d. -f3 | cut -d- -f1)

# Increment based on type
case $INCREMENT_TYPE in
    major)
        NEW_VERSION="v$((MAJOR + 1)).0.0"
        echo "Breaking change: $MAJOR.$MINOR.$PATCH → $NEW_VERSION"
        ;;
    minor)
        NEW_VERSION="v$MAJOR.$((MINOR + 1)).0"
        echo "New feature: $MAJOR.$MINOR.$PATCH → $NEW_VERSION"
        ;;
    patch)
        NEW_VERSION="v$MAJOR.$MINOR.$((PATCH + 1))"
        echo "Bug fix: $MAJOR.$MINOR.$PATCH → $NEW_VERSION"
        ;;
    *)
        echo "Usage: /git/version/increment major|minor|patch"
        exit 1
        ;;
esac

echo ""
echo "Next steps:"
echo "1. Create release branch: git checkout -b release/$NEW_VERSION"
echo "2. Or tag directly: git tag -a $NEW_VERSION -m \"Release $NEW_VERSION\""
```