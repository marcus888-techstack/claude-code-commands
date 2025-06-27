Complete feature and merge to develop with automatic version tagging

```bash
# Get feature name from arguments
FEATURE_NAME=$ARGUMENTS

if [ -z "$FEATURE_NAME" ]; then
    echo "Usage: /git:feature:finish <feature-name>"
    exit 1
fi

BRANCH_NAME="feature/$FEATURE_NAME"

# Verify feature branch exists
if ! git show-ref --verify --quiet "refs/heads/$BRANCH_NAME"; then
    echo "Error: Branch '$BRANCH_NAME' not found"
    exit 1
fi

# Switch to develop and update
git checkout develop
git pull origin develop

# Get current develop version for auto-tagging
CURRENT_VERSION=$(git describe --tags --abbrev=0 develop 2>/dev/null | sed 's/_dev$//')
if [ -z "$CURRENT_VERSION" ]; then
    echo "Warning: No version tags found on develop"
    CURRENT_VERSION="v0.0.0"
fi

# Parse and increment patch version
MAJOR=$(echo $CURRENT_VERSION | cut -d. -f1 | sed 's/v//')
MINOR=$(echo $CURRENT_VERSION | cut -d. -f2)
PATCH=$(echo $CURRENT_VERSION | cut -d. -f3)
NEW_PATCH=$((PATCH + 1))
NEW_VERSION="v${MAJOR}.${MINOR}.${NEW_PATCH}_dev"

# Merge feature branch
echo "Merging $BRANCH_NAME into develop..."
git merge --no-ff "$BRANCH_NAME" -m "Merge $BRANCH_NAME into develop

Feature: $FEATURE_NAME completed"

if [ $? -eq 0 ]; then
    # Push develop
    git push origin develop
    
    # Create development version tag
    git tag -a "$NEW_VERSION" -m "Feature merged: $FEATURE_NAME"
    git push origin "$NEW_VERSION"
    
    # Delete feature branch
    git branch -d "$BRANCH_NAME"
    git push origin --delete "$BRANCH_NAME"
    
    # Remove worktree if exists
    WORKTREE_PATH="../features/$FEATURE_NAME"
    if [ -d "$WORKTREE_PATH" ]; then
        git worktree remove "$WORKTREE_PATH" 2>/dev/null || true
    fi
    
    echo ""
    echo "✓ Feature '$FEATURE_NAME' completed successfully"
    echo "✓ Merged to develop"
    echo "✓ Tagged as $NEW_VERSION"
    echo "✓ Branch cleaned up"
else
    echo "Error: Merge failed. Resolve conflicts and try again."
    exit 1
fi
```