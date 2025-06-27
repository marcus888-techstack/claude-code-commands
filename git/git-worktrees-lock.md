Lock or unlock a worktree to prevent accidental removal

```bash
# Parse action and worktree path from arguments
ARGS=$ARGUMENTS
ACTION=$(echo $ARGS | cut -d' ' -f1)
WORKTREE_PATH=$(echo $ARGS | cut -d' ' -f2-)

if [ -z "$ACTION" ]; then
    echo "Usage: /git:worktrees:lock <lock|unlock|status> [worktree-path]"
    echo ""
    echo "Actions:"
    echo "  lock <path>    - Lock a worktree"
    echo "  unlock <path>  - Unlock a worktree"
    echo "  status         - Show lock status of all worktrees"
    echo ""
    echo "Examples:"
    echo "  /git:worktrees:lock lock ../features/critical"
    echo "  /git:worktrees:lock unlock ../features/critical"
    echo "  /git:worktrees:lock status"
    exit 1
fi

case "$ACTION" in
    "lock")
        if [ -z "$WORKTREE_PATH" ]; then
            echo "Error: Please specify worktree path to lock"
            exit 1
        fi
        
        # Verify worktree exists
        if ! git worktree list | grep -q "$WORKTREE_PATH"; then
            echo "Error: Worktree not found: $WORKTREE_PATH"
            echo ""
            echo "Available worktrees:"
            git worktree list
            exit 1
        fi
        
        # Lock the worktree
        git worktree lock "$WORKTREE_PATH"
        
        if [ $? -eq 0 ]; then
            echo "✓ Locked worktree: $WORKTREE_PATH"
            echo ""
            echo "This worktree is now protected from:"
            echo "- Accidental removal with 'git worktree remove'"
            echo "- Automatic pruning"
            echo ""
            echo "To unlock: /git:worktrees:lock unlock $WORKTREE_PATH"
        else
            echo "Error: Failed to lock worktree"
        fi
        ;;
        
    "unlock")
        if [ -z "$WORKTREE_PATH" ]; then
            echo "Error: Please specify worktree path to unlock"
            exit 1
        fi
        
        # Unlock the worktree
        git worktree unlock "$WORKTREE_PATH"
        
        if [ $? -eq 0 ]; then
            echo "✓ Unlocked worktree: $WORKTREE_PATH"
            echo ""
            echo "This worktree can now be removed normally"
        else
            echo "Error: Failed to unlock worktree"
            echo "The worktree might not be locked"
        fi
        ;;
        
    "status")
        echo "=== Worktree Lock Status ==="
        echo ""
        
        # Check each worktree for lock status
        git worktree list --porcelain | awk '
            /^worktree/ {path=$2}
            /^locked/ {locked="Yes"; reason=$2}
            /^$/ {
                if (path) {
                    status = locked ? "🔒 Locked" : "🔓 Unlocked"
                    printf "%-40s %s", path, status
                    if (reason) printf " (%s)", reason
                    printf "\n"
                }
                path=""; locked=""; reason=""
            }
        '
        
        echo ""
        echo "Legend:"
        echo "🔒 Locked   - Protected from removal"
        echo "🔓 Unlocked - Can be removed"
        ;;
        
    *)
        echo "Error: Unknown action '$ACTION'"
        echo "Use: lock, unlock, or status"
        exit 1
        ;;
esac
```