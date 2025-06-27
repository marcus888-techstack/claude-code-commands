Stash and manage work in progress changes

```bash
# Parse stash command from arguments
COMMAND=$ARGUMENTS

if [ -z "$COMMAND" ]; then
    # Default: create stash
    git stash push -m "WIP: $(date +%Y-%m-%d-%H%M%S)"
    echo "✓ Changes stashed"
elif [[ "$COMMAND" =~ ^save ]]; then
    # Save with message
    MESSAGE=$(echo $COMMAND | sed 's/save //')
    git stash push -m "$MESSAGE"
    echo "✓ Stashed: $MESSAGE"
elif [ "$COMMAND" = "list" ]; then
    # List stashes
    echo "=== Stashes ==="
    git stash list
elif [ "$COMMAND" = "pop" ]; then
    # Apply and remove latest stash
    git stash pop
    echo "✓ Applied and removed latest stash"
elif [[ "$COMMAND" =~ ^apply ]]; then
    # Apply specific stash
    STASH_REF=$(echo $COMMAND | awk '{print $2}')
    if [ -z "$STASH_REF" ]; then
        git stash apply
    else
        git stash apply "$STASH_REF"
    fi
    echo "✓ Applied stash"
elif [ "$COMMAND" = "clear" ]; then
    # Clear all stashes
    git stash clear
    echo "✓ Cleared all stashes"
else
    echo "Usage: /git/utility/stash [save message|list|pop|apply [ref]|clear]"
fi
```