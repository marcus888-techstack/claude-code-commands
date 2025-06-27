View formatted git history with various options

```bash
# Parse log options from arguments
OPTIONS=$ARGUMENTS

if [ -z "$OPTIONS" ]; then
    # Default: pretty log with graph
    git log --oneline --graph --decorate --all -20
elif [ "$OPTIONS" = "feature" ]; then
    # Show feature branch history
    git log --oneline --graph --first-parent develop
elif [ "$OPTIONS" = "release" ]; then
    # Show release history
    git log --oneline --decorate --tags --no-walk
elif [ "$OPTIONS" = "author" ]; then
    # Group by author
    git shortlog -sn --all
elif [ "$OPTIONS" = "files" ]; then
    # Show file changes
    git log --stat --oneline -10
elif [ "$OPTIONS" = "search" ]; then
    # Search commit messages
    echo "Enter search term:"
    read SEARCH_TERM
    git log --grep="$SEARCH_TERM" --oneline
elif [ "$OPTIONS" = "between" ]; then
    # Commits between tags
    echo "Enter two tags (e.g., v1.0 v2.0):"
    read TAG1 TAG2
    git log --oneline "$TAG1".."$TAG2"
else
    # Custom format
    git log $OPTIONS
fi
```