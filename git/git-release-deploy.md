Deploy from a tagged release

```bash
# Parse tag/version from arguments
TAG=$ARGUMENTS

if [ -z "$TAG" ]; then
    # Show recent tags to choose from
    echo "=== Recent Release Tags ==="
    git tag -l --sort=-version:refname | grep -E "^v[0-9]+\.[0-9]+\.[0-9]+$" | head -10
    echo ""
    echo "Usage: /git/release/deploy v1.2.0"
    exit 1
fi

# Ensure tag format
if [[ ! "$TAG" =~ ^v ]]; then
    TAG="v$TAG"
fi

# Verify tag exists
if ! git rev-parse "$TAG" >/dev/null 2>&1; then
    echo "Error: Tag '$TAG' not found"
    echo "Available tags:"
    git tag -l --sort=-version:refname | grep -E "^v[0-9]" | head -10
    exit 1
fi

echo "=== Deploying Release: $TAG ==="
echo ""

# Show tag information
git show "$TAG" --no-patch
echo ""

# Checkout the tag in a deployment worktree (if using worktrees)
if command -v git worktree &> /dev/null; then
    DEPLOY_DIR="../deploy-$TAG"
    if [ -d "$DEPLOY_DIR" ]; then
        echo "Deployment directory already exists: $DEPLOY_DIR"
        cd "$DEPLOY_DIR"
        git status
    else
        git worktree add "$DEPLOY_DIR" "$TAG"
        cd "$DEPLOY_DIR"
        echo "✓ Created deployment worktree at $DEPLOY_DIR"
    fi
else
    # Traditional checkout
    git checkout "$TAG"
    echo "✓ Checked out tag $TAG"
fi

echo ""
echo "=== Pre-deployment Checklist ==="
echo "[ ] Build application"
echo "[ ] Run production tests"
echo "[ ] Update environment variables"
echo "[ ] Backup database"
echo "[ ] Notify team"
echo ""

# Example deployment commands (customize as needed)
echo "Run deployment commands:"
echo "  npm run build       # or: yarn build"
echo "  npm run deploy      # or: make deploy"
echo "  docker build -t app:$TAG ."
echo "  kubectl set image deployment/app app=app:$TAG"
```