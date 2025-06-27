Squash merge commits into a single commit

```bash
# Parse branch name from arguments
BRANCH=$ARGUMENTS

if [ -z "$BRANCH" ]; then
    echo "Error: Please specify branch to merge"
    echo "Usage: /git/merge/squash branch-name"
    exit 1
fi

# Get current branch
CURRENT=$(git branch --show-current)

# Perform squash merge
git merge --squash $BRANCH

echo "✓ Changes from $BRANCH staged for commit"
echo ""
echo "Staged changes:"
git status --short

echo ""
echo "Create a single commit with:"
echo "git commit -m \"Squashed changes from $BRANCH\""
```