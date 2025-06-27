Create a feature branch worktree

```bash
# Parse feature name from arguments
FEATURE_NAME=$ARGUMENTS

if [ -z "$FEATURE_NAME" ]; then
    echo "Usage: /git/worktrees/feature <feature-name>"
    echo ""
    echo "Examples:"
    echo "  /git/worktrees/feature user-auth"
    echo "  /git/worktrees/feature payment-integration"
    echo "  /git/worktrees/feature issue-123-fix-login"
    exit 1
fi

# Ensure we're starting from develop
git checkout develop > /dev/null 2>&1
git pull origin develop > /dev/null 2>&1

# Create branch name
BRANCH_NAME="feature/$FEATURE_NAME"
WORKTREE_PATH="../features/$FEATURE_NAME"

# Check if branch already exists
if git show-ref --verify --quiet "refs/heads/$BRANCH_NAME"; then
    echo "Branch '$BRANCH_NAME' already exists"
    echo "Creating worktree from existing branch..."
    git worktree add "$WORKTREE_PATH" "$BRANCH_NAME"
else
    echo "Creating new feature branch: $BRANCH_NAME"
    git worktree add -b "$BRANCH_NAME" "$WORKTREE_PATH"
fi

# Set up feature worktree
cd "$WORKTREE_PATH"

# Create feature-specific files if needed
if [ ! -f ".feature-info" ]; then
    cat > .feature-info << EOF
Feature: $FEATURE_NAME
Branch: $BRANCH_NAME
Created: $(date)
Author: $(git config user.name)

Description:
TODO: Add feature description here

Tasks:
[ ] Implement core functionality
[ ] Write tests
[ ] Update documentation
[ ] Code review
EOF
    echo "✓ Created .feature-info file"
fi

echo ""
echo "=== Feature Worktree Created ==="
echo "Path: $(pwd)"
echo "Branch: $BRANCH_NAME"
echo ""
echo "Next steps:"
echo "1. cd $WORKTREE_PATH"
echo "2. Edit .feature-info with feature details"
echo "3. Start development"
echo "4. When done: /git/feature/finish $FEATURE_NAME"

# Return to original directory
cd - > /dev/null
```