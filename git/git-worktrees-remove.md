Remove a Git worktree safely

```bash
# Parse worktree path from arguments
WORKTREE_PATH=$ARGUMENTS

if [ -z "$WORKTREE_PATH" ]; then
    echo "Usage: /git/worktrees/remove <worktree-path>"
    echo ""
    echo "Current worktrees:"
    git worktree list
    exit 1
fi

# Check if worktree exists
if ! git worktree list | grep -q "$WORKTREE_PATH"; then
    echo "Error: Worktree not found: $WORKTREE_PATH"
    echo ""
    echo "Available worktrees:"
    git worktree list
    exit 1
fi

# Get branch name for the worktree
BRANCH=$(git worktree list --porcelain | grep -A2 "^worktree $WORKTREE_PATH" | grep "^branch" | cut -d' ' -f2 | sed 's|refs/heads/||')

# Check for uncommitted changes
echo "Checking worktree status..."
cd "$WORKTREE_PATH" 2>/dev/null
if [ $? -eq 0 ]; then
    if [ -n "$(git status --porcelain)" ]; then
        echo "⚠️  Warning: Worktree has uncommitted changes:"
        git status --short
        echo ""
        echo "Options:"
        echo "1. Commit changes first"
        echo "2. Stash changes: git stash"
        echo "3. Force remove (loses changes): git worktree remove --force $WORKTREE_PATH"
        exit 1
    fi
    cd - > /dev/null
fi

# Check if branch is merged
if [ -n "$BRANCH" ]; then
    MERGED_TO_DEVELOP=$(git branch --merged develop | grep -c "$BRANCH" || echo 0)
    MERGED_TO_MAIN=$(git branch --merged main | grep -c "$BRANCH" || echo 0)
    
    if [ $MERGED_TO_DEVELOP -eq 0 ] && [ $MERGED_TO_MAIN -eq 0 ]; then
        echo "⚠️  Warning: Branch '$BRANCH' is not merged to develop or main"
        echo "Continue anyway? (yes/no)"
        read CONFIRM
        if [ "$CONFIRM" != "yes" ]; then
            echo "Removal cancelled"
            exit 0
        fi
    fi
fi

# Remove worktree
git worktree remove "$WORKTREE_PATH"

if [ $? -eq 0 ]; then
    echo "✓ Removed worktree: $WORKTREE_PATH"
    
    # Offer to delete branch if merged
    if [ -n "$BRANCH" ]; then
        if [ $MERGED_TO_DEVELOP -gt 0 ] || [ $MERGED_TO_MAIN -gt 0 ]; then
            echo ""
            echo "Delete merged branch '$BRANCH'? (yes/no)"
            read DELETE_BRANCH
            if [ "$DELETE_BRANCH" = "yes" ]; then
                git branch -d "$BRANCH"
                echo "✓ Deleted branch: $BRANCH"
            fi
        fi
    fi
else
    echo "Error: Failed to remove worktree"
    echo "Try: git worktree remove --force $WORKTREE_PATH"
fi

# Show remaining worktrees
echo ""
echo "Remaining worktrees:"
git worktree list
```