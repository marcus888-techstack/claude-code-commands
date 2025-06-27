Switch between Git worktrees quickly

```bash
# Parse worktree name/path from arguments
TARGET=$ARGUMENTS

if [ -z "$TARGET" ]; then
    echo "=== Available Worktrees ==="
    echo ""
    
    # List worktrees with current indicator
    git worktree list --porcelain | awk '
        /^worktree/ {path=$2; gsub(/.*\//, "", name, path)}
        /^HEAD/ {head=substr($2, 1, 7)}
        /^branch/ {branch=$2; gsub("refs/heads/", "", branch)}
        /^$/ {
            current = (path == ENVIRON["PWD"]) ? " *" : ""
            printf "%-20s %-30s %s%s\n", name, branch ? branch : "(detached)", head, current
            path=""; head=""; branch=""
        }
    ' | column -t
    
    echo ""
    echo "Usage: /git:worktrees:switch <name>"
    echo ""
    echo "Examples:"
    echo "  /git:worktrees:switch develop"
    echo "  /git:worktrees:switch feature/oauth"
    echo "  /git:worktrees:switch hotfix/security"
    exit 0
fi

# Find worktree path
WORKTREE_PATH=""

# Check if it's a direct path
if [ -d "$TARGET" ]; then
    WORKTREE_PATH="$TARGET"
else
    # Search in common locations
    for BASE in .. ../features ../hotfixes .; do
        if [ -d "$BASE/$TARGET" ]; then
            WORKTREE_PATH="$BASE/$TARGET"
            break
        fi
    done
    
    # Search by branch name
    if [ -z "$WORKTREE_PATH" ]; then
        BRANCH_WORKTREE=$(git worktree list --porcelain | grep -B1 "branch.*$TARGET" | grep "^worktree" | cut -d' ' -f2)
        if [ ! -z "$BRANCH_WORKTREE" ]; then
            WORKTREE_PATH="$BRANCH_WORKTREE"
        fi
    fi
fi

if [ -z "$WORKTREE_PATH" ] || [ ! -d "$WORKTREE_PATH" ]; then
    echo "Error: Worktree not found: $TARGET"
    echo ""
    echo "Available worktrees:"
    git worktree list | awk '{print "  " $1}'
    exit 1
fi

# Switch to worktree
cd "$WORKTREE_PATH"
echo "✓ Switched to worktree: $(pwd)"
echo "✓ Branch: $(git branch --show-current)"

# Show status
echo ""
git status -sb

# Start new shell in worktree
exec $SHELL
```