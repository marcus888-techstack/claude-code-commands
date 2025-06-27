Tag feature merge in develop branch with incremented patch version

```bash
# Get current develop version
CURRENT_VERSION=$(git describe --tags --abbrev=0 develop | sed 's/_dev$//')

# Parse version components
MAJOR=$(echo $CURRENT_VERSION | cut -d. -f1 | sed 's/v//')
MINOR=$(echo $CURRENT_VERSION | cut -d. -f2)
PATCH=$(echo $CURRENT_VERSION | cut -d. -f3)

# Increment patch version
NEW_PATCH=$((PATCH + 1))
NEW_VERSION="v${MAJOR}.${MINOR}.${NEW_PATCH}_dev"

# Create tag with feature description
git tag -a "$NEW_VERSION" -m "Feature: $ARGUMENTS"
git push origin "$NEW_VERSION"

echo "✓ Tagged feature in develop as $NEW_VERSION"
echo "✓ Description: $ARGUMENTS"
```