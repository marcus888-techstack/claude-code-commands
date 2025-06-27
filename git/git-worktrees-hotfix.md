Create a hotfix branch worktree for urgent production fixes

```bash
# Parse hotfix name from arguments
HOTFIX_NAME=$ARGUMENTS

if [ -z "$HOTFIX_NAME" ]; then
    echo "Usage: /git/worktrees/hotfix <hotfix-name>"
    echo ""
    echo "Examples:"
    echo "  /git/worktrees/hotfix security-patch"
    echo "  /git/worktrees/hotfix critical-bug-fix"
    echo "  /git/worktrees/hotfix cve-2024-001"
    exit 1
fi

# Ensure we're starting from main
git checkout main > /dev/null 2>&1
git pull origin main > /dev/null 2>&1

# Get current production version
CURRENT_VERSION=$(git describe --tags --abbrev=0 main 2>/dev/null || echo "v0.0.0")
echo "Creating hotfix from production version: $CURRENT_VERSION"

# Create branch name
BRANCH_NAME="hotfix/$HOTFIX_NAME"
WORKTREE_PATH="../hotfixes/$HOTFIX_NAME"

# Check if branch already exists
if git show-ref --verify --quiet "refs/heads/$BRANCH_NAME"; then
    echo "Error: Branch '$BRANCH_NAME' already exists"
    exit 1
fi

# Create hotfix worktree
echo "Creating hotfix branch: $BRANCH_NAME"
git worktree add -b "$BRANCH_NAME" "$WORKTREE_PATH"

# Set up hotfix worktree
cd "$WORKTREE_PATH"

# Create hotfix tracking file
cat > .hotfix-info << EOF
Hotfix: $HOTFIX_NAME
Branch: $BRANCH_NAME
Base Version: $CURRENT_VERSION
Created: $(date)
Author: $(git config user.name)

Issue:
TODO: Describe the critical issue being fixed

Solution:
TODO: Describe the fix approach

Testing:
[ ] Fix implemented
[ ] Tests added/updated
[ ] Manually tested
[ ] Ready for deployment
EOF

echo "✓ Created .hotfix-info file"

echo ""
echo "=== Hotfix Worktree Created ==="
echo "Path: $(pwd)"
echo "Branch: $BRANCH_NAME"
echo "Base: $CURRENT_VERSION"
echo ""
echo "⚠️  HOTFIX WORKFLOW:"
echo "1. cd $WORKTREE_PATH"
echo "2. Implement critical fix"
echo "3. Test thoroughly"
echo "4. When done: /git/hotfix/finish $HOTFIX_NAME"
echo ""
echo "This will:"
echo "- Merge to main with patch version"
echo "- Apply fix to develop"
echo "- Create deployment tag"

# Return to original directory
cd - > /dev/null
```