Synchronize branches after tag-based releases or hotfixes

```bash
# Determine sync type from arguments
SYNC_TYPE=$ARGUMENTS

if [ -z "$SYNC_TYPE" ]; then
    echo "Usage: /git:workflow:sync release|hotfix|status"
    echo ""
    echo "Options:"
    echo "  release - Sync develop after a release tag"
    echo "  hotfix  - Sync develop after a hotfix"
    echo "  status  - Show sync status between branches"
    exit 1
fi

if [ "$SYNC_TYPE" = "status" ]; then
    # Show sync status
    echo "=== Branch Synchronization Status ==="
    echo ""
    
    MAIN_VERSION=$(git describe --tags --abbrev=0 main 2>/dev/null || echo "none")
    DEVELOP_VERSION=$(git describe --tags --abbrev=0 develop 2>/dev/null || echo "none")
    
    echo "Main version: $MAIN_VERSION"
    echo "Develop version: $DEVELOP_VERSION"
    echo ""
    
    # Check divergence
    AHEAD=$(git rev-list --count main..develop)
    BEHIND=$(git rev-list --count develop..main)
    
    echo "Develop is $AHEAD commits ahead of main"
    echo "Develop is $BEHIND commits behind main"
    
    if [ $BEHIND -gt 0 ]; then
        echo ""
        echo "⚠️  Develop needs to be synchronized with main"
        echo "Run: /git:workflow:sync release"
    else
        echo ""
        echo "✓ Branches are synchronized"
    fi
    
elif [ "$SYNC_TYPE" = "release" ]; then
    # After release: sync develop with main
    echo "Synchronizing develop with main after release..."
    
    git checkout main
    git pull origin main
    MAIN_VERSION=$(git describe --tags --abbrev=0)
    
    git checkout develop
    git pull origin develop
    
    # Only merge if needed
    if ! git merge-base --is-ancestor main develop; then
        git merge main --no-ff -m "Sync: develop aligned with release $MAIN_VERSION"
        git push origin develop
    fi
    
    # Create dev tag for the release version
    DEV_TAG="${MAIN_VERSION}_dev"
    if ! git rev-parse "$DEV_TAG" >/dev/null 2>&1; then
        git tag -a "$DEV_TAG" -m "Development continues from $MAIN_VERSION"
        git push origin "$DEV_TAG"
    fi
    
    echo "✓ Develop synchronized with main at $MAIN_VERSION"
    
elif [ "$SYNC_TYPE" = "hotfix" ]; then
    # After hotfix: ensure both branches have the fix
    echo "Synchronizing develop with hotfix from main..."
    
    HOTFIX_VERSION=$(git describe --tags --abbrev=0 main)
    
    git checkout develop
    git pull origin develop
    git merge main --no-ff -m "Sync: hotfix $HOTFIX_VERSION applied to develop"
    git push origin develop
    
    # Tag develop with hotfix version
    DEV_TAG="${HOTFIX_VERSION}_dev"
    if ! git rev-parse "$DEV_TAG" >/dev/null 2>&1; then
        git tag -a "$DEV_TAG" -m "Hotfix $HOTFIX_VERSION synchronized"
        git push origin "$DEV_TAG"
    fi
    
    echo "✓ Hotfix $HOTFIX_VERSION synchronized to develop"
else
    echo "Error: Unknown sync type '$SYNC_TYPE'"
    echo "Use: release, hotfix, or status"
fi
```