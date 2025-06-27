Reset repository to a specific state

```bash
# Parse reset type and target from arguments
ARGS=$ARGUMENTS
TYPE=$(echo $ARGS | cut -d' ' -f1)
TARGET=$(echo $ARGS | cut -d' ' -f2-)

if [ -z "$TYPE" ]; then
    echo "Usage: /git/utility/reset [soft|mixed|hard] [commit|HEAD~n]"
    echo ""
    echo "Options:"
    echo "  soft:  Keep changes staged"
    echo "  mixed: Keep changes unstaged (default)"
    echo "  hard:  Discard all changes"
    echo ""
    echo "Examples:"
    echo "  /git/utility/reset soft HEAD~1"
    echo "  /git/utility/reset hard origin/main"
    exit 1
fi

# Default target is HEAD~1
if [ -z "$TARGET" ]; then
    TARGET="HEAD~1"
fi

# Confirm for hard reset
if [ "$TYPE" = "hard" ]; then
    echo "⚠️  WARNING: This will discard all local changes!"
    echo "Target: $TARGET"
    echo "Continue? (yes/no)"
    read CONFIRM
    if [ "$CONFIRM" != "yes" ]; then
        echo "Reset cancelled"
        exit 0
    fi
fi

# Perform reset
git reset --$TYPE $TARGET

echo "✓ Reset complete ($TYPE to $TARGET)"
echo ""
git status --short
```