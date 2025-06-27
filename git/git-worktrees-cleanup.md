Clean up worktrees for merged branches

```bash
# Optional filter from arguments
FILTER=$ARGUMENTS

echo "=== Worktree Cleanup ==="
echo ""

# Get list of worktrees
WORKTREES=$(git worktree list --porcelain | grep "^worktree" | cut -d' ' -f2)
CLEANED=0
SKIPPED=0

for WORKTREE in $WORKTREES; do
    # Skip main worktree
    if git -C "$WORKTREE" symbolic-ref --short HEAD >/dev/null 2>&1; then
        BRANCH=$(git -C "$WORKTREE" symbolic-ref --short HEAD)
        
        # Skip main and develop branches
        if [[ "$BRANCH" = "main" ]] || [[ "$BRANCH" = "develop" ]]; then
            continue
        fi
        
        # Apply filter if provided
        if [ ! -z "$FILTER" ] && [[ ! "$BRANCH" =~ $FILTER ]]; then
            continue
        fi
        
        # Check if branch is merged to develop or main
        MERGED_DEVELOP=$(git branch --merged develop | grep -c "^  $BRANCH$" || echo 0)
        MERGED_MAIN=$(git branch --merged main | grep -c "^  $BRANCH$" || echo 0)
        
        if [ $MERGED_DEVELOP -gt 0 ] || [ $MERGED_MAIN -gt 0 ]; then
            echo "Cleaning up merged worktree: $WORKTREE ($BRANCH)"
            
            # Check for uncommitted changes
            if [ -z "$(git -C "$WORKTREE" status --porcelain)" ]; then
                git worktree remove "$WORKTREE"
                
                # Delete the branch
                git branch -d "$BRANCH" 2>/dev/null
                git push origin --delete "$BRANCH" 2>/dev/null
                
                CLEANED=$((CLEANED + 1))
                echo "✓ Removed worktree and deleted branch: $BRANCH"
            else
                echo "⚠️  Skipped: $WORKTREE has uncommitted changes"
                SKIPPED=$((SKIPPED + 1))
            fi
        fi
    fi
done

# Prune any stale worktree entries
git worktree prune

echo ""
echo "=== Cleanup Summary ==="
echo "Cleaned: $CLEANED worktrees"
echo "Skipped: $SKIPPED worktrees (uncommitted changes)"

# Show remaining worktrees
echo ""
echo "Remaining worktrees:"
git worktree list

# Suggest next steps
if [ $SKIPPED -gt 0 ]; then
    echo ""
    echo "To force cleanup skipped worktrees:"
    echo "git worktree remove --force <worktree-path>"
fi
```