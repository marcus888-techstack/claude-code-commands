Create pre-release tags (alpha/beta/rc)

```bash
# Parse arguments: "version type [number]"
# Example: "v2.0.0 beta 1" or "v2.0.0 alpha"
VERSION=$(echo $ARGUMENTS | cut -d' ' -f1)
TYPE=$(echo $ARGUMENTS | cut -d' ' -f2)
NUMBER=$(echo $ARGUMENTS | cut -d' ' -f3)

# Validate inputs
if [[ ! "$VERSION" =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
    echo "Error: Invalid version format. Use: vX.Y.Z"
    exit 1
fi

if [[ ! "$TYPE" =~ ^(alpha|beta|rc)$ ]]; then
    echo "Error: Invalid pre-release type. Use: alpha, beta, or rc"
    exit 1
fi

# Build pre-release tag
if [ -z "$NUMBER" ]; then
    PRERELEASE_TAG="${VERSION}-${TYPE}"
else
    PRERELEASE_TAG="${VERSION}-${TYPE}.${NUMBER}"
fi

# Create and push tag
git tag -a "$PRERELEASE_TAG" -m "Pre-release: $PRERELEASE_TAG"
git push origin "$PRERELEASE_TAG"

echo "✓ Created pre-release tag: $PRERELEASE_TAG"
echo ""
echo "Pre-release sequence:"
echo "  ${VERSION}-alpha → ${VERSION}-beta → ${VERSION}-rc → ${VERSION}"
```