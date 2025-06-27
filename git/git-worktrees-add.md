Add a new Git worktree

```bash
# Parse arguments: "path branch" or "path -b new-branch"
ARGS=$ARGUMENTS
PATH_ARG=$(echo $ARGS | awk '{print $1}')
BRANCH_ARGS=$(echo $ARGS | cut -d' ' -f2-)

if [ -z "$PATH_ARG" ]; then
    echo "Usage: /git/worktrees/add <path> <branch>"
    echo "       /git/worktrees/add <path> -b <new-branch>"
    echo ""
    echo "Examples:"
    echo "  /git/worktrees/add ../features/oauth feature/oauth"
    echo "  /git/worktrees/add ../features/payment -b feature/payment"
    exit 1
fi

# Determine if creating new branch
if [[ "$BRANCH_ARGS" =~ ^-b ]]; then
    # Creating new branch
    BRANCH_NAME=$(echo $BRANCH_ARGS | sed 's/-b //')
    git worktree add -b "$BRANCH_NAME" "$PATH_ARG"
    echo "✓ Created worktree at $PATH_ARG with new branch: $BRANCH_NAME"
else
    # Using existing branch
    BRANCH_NAME=$BRANCH_ARGS
    if [ -z "$BRANCH_NAME" ]; then
        echo "Error: Please specify branch name"
        exit 1
    fi
    git worktree add "$PATH_ARG" "$BRANCH_NAME"
    echo "✓ Created worktree at $PATH_ARG for branch: $BRANCH_NAME"
fi

# Show worktree info
echo ""
echo "Worktree details:"
echo "- Path: $(cd "$PATH_ARG" && pwd)"
echo "- Branch: $BRANCH_NAME"
echo "- Status: Active"

# Provide next steps
echo ""
echo "Next steps:"
echo "1. cd $PATH_ARG"
echo "2. Start development"
echo "3. Remove when done: git worktree remove $PATH_ARG"
```