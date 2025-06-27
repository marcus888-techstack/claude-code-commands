Create a release using tags (modern workflow without release branches)

```bash
# Parse version from arguments
VERSION=$ARGUMENTS

if [ -z "$VERSION" ]; then
    echo "Error: Please specify version"
    echo "Usage: /git/release/tag v1.2.0"
    exit 1
fi

# Ensure version format
if [[ ! "$VERSION" =~ ^v ]]; then
    VERSION="v$VERSION"
fi

# Current branch check
CURRENT_BRANCH=$(git branch --show-current)

echo "=== Tag-Based Release: $VERSION ==="
echo ""

# Option 1: Tag from develop and merge to main
if [ "$CURRENT_BRANCH" = "develop" ]; then
    # Ensure develop is up to date
    git pull origin develop
    
    # Run tests (customize as needed)
    echo "Running tests..."
    # npm test || yarn test || make test
    
    # Create release tag
    git tag -a "$VERSION" -m "Release $VERSION"
    echo "✓ Tagged develop with $VERSION"
    
    # Push tag
    git push origin "$VERSION"
    echo "✓ Pushed tag to origin"
    
    # Merge to main
    echo ""
    echo "Merging to main..."
    git checkout main
    git pull origin main
    git merge develop --no-ff -m "Release $VERSION"
    git push origin main
    echo "✓ Merged to main"
    
    # Update develop version
    git checkout develop
    git tag -a "${VERSION}_dev" -m "Development continues from $VERSION"
    git push origin "${VERSION}_dev"
    echo "✓ Tagged develop as ${VERSION}_dev"
    
# Option 2: Tag directly from main
elif [ "$CURRENT_BRANCH" = "main" ]; then
    # Ensure main is up to date
    git pull origin main
    
    # Create release tag
    git tag -a "$VERSION" -m "Release $VERSION"
    git push origin "$VERSION"
    echo "✓ Tagged main with $VERSION"
    
    # Sync develop
    git checkout develop
    git pull origin develop
    git merge main --no-ff -m "Sync with release $VERSION"
    git tag -a "${VERSION}_dev" -m "Development synchronized with $VERSION"
    git push origin develop
    git push origin "${VERSION}_dev"
    echo "✓ Synchronized develop"
else
    echo "Error: Must be on 'develop' or 'main' branch"
    echo "Current branch: $CURRENT_BRANCH"
    exit 1
fi

echo ""
echo "=== Release Complete ==="
echo "Version: $VERSION"
echo "Deploy using: git checkout $VERSION"
echo ""
echo "Next steps:"
echo "1. Deploy from tag: $VERSION"
echo "2. Create release notes"
echo "3. Notify team"
```