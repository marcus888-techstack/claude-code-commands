Merge branch with --no-ff to preserve branch history

```bash
# Parse branch name from arguments
BRANCH=$ARGUMENTS

if [ -z "$BRANCH" ]; then
    echo "Error: Please specify branch to merge"
    echo "Usage: /git/merge/no-ff branch-name"
    exit 1
fi

# Get current branch
CURRENT=$(git branch --show-current)

# Perform no-fast-forward merge
git merge --no-ff $BRANCH -m "Merge branch '$BRANCH' into $CURRENT

This merge preserves the branch history and creates a merge commit
even if a fast-forward merge is possible."

echo "✓ Merged $BRANCH into $CURRENT with merge commit"
echo ""
echo "Merge commit created:"
git log -1 --oneline
```