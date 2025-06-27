Help resolve merge conflicts step by step

```bash
# Check for merge conflicts
CONFLICTS=$(git diff --name-only --diff-filter=U)

if [ -z "$CONFLICTS" ]; then
    echo "No merge conflicts detected"
    exit 0
fi

echo "=== Merge Conflict Resolution ==="
echo ""
echo "Conflicted files:"
echo "$CONFLICTS" | nl

echo ""
echo "Resolution options:"
echo "1. Accept current branch (ours): git checkout --ours <file>"
echo "2. Accept incoming branch (theirs): git checkout --theirs <file>"
echo "3. Manual edit: Open file and resolve conflicts"
echo ""

# If specific file provided as argument
if [ ! -z "$ARGUMENTS" ]; then
    FILE=$ARGUMENTS
    echo "Showing conflict markers in $FILE:"
    echo ""
    grep -n "^<<<<<<< \|^======= \|^>>>>>>> " "$FILE" || echo "No conflict markers found"
    
    echo ""
    echo "Next steps for $FILE:"
    echo "- Edit the file to resolve conflicts"
    echo "- git add $FILE"
    echo "- Continue merge: git commit"
else
    echo "To resolve all conflicts:"
    echo "1. Edit each file to resolve conflicts"
    echo "2. git add <resolved-files>"
    echo "3. git commit to complete merge"
fi
```