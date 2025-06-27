List all Git worktrees with enhanced information

```bash
# Optional filter from arguments
FILTER=$ARGUMENTS

echo "=== Git Worktrees ==="
echo ""

# Get worktree information
if [ -z "$FILTER" ]; then
    # Full listing
    git worktree list --porcelain | awk '
        /^worktree/ {path=$2}
        /^HEAD/ {head=$2}
        /^branch/ {branch=$2; gsub("refs/heads/", "", branch)}
        /^$/ {
            if (path && head) {
                # Get relative path if possible
                gsub(/.*\//, "", shortpath)
                printf "%-30s %-25s %s\n", path, branch ? branch : "(detached)", substr(head, 1, 7)
            }
            path=""; head=""; branch=""
        }
    ' | column -t
else
    # Filtered listing
    git worktree list | grep "$FILTER"
fi

# Summary statistics
echo ""
echo "=== Summary ==="
TOTAL=$(git worktree list | wc -l)
FEATURES=$(git worktree list | grep -c "feature/" || echo 0)
HOTFIXES=$(git worktree list | grep -c "hotfix/" || echo 0)

echo "Total worktrees: $TOTAL"
echo "- Main/Develop: $((TOTAL - FEATURES - HOTFIXES))"
echo "- Features: $FEATURES"
echo "- Hotfixes: $HOTFIXES"

# Check for stale worktrees
STALE=$(git worktree list --porcelain | grep -B2 "prunable" | grep "^worktree" | wc -l)
if [ $STALE -gt 0 ]; then
    echo ""
    echo "⚠️  Found $STALE stale worktree(s)"
    echo "Run 'git worktree prune' to clean up"
fi

# Disk usage
if command -v du &> /dev/null; then
    echo ""
    echo "=== Disk Usage ==="
    for worktree in $(git worktree list --porcelain | grep "^worktree" | cut -d' ' -f2); do
        if [ -d "$worktree" ]; then
            SIZE=$(du -sh "$worktree" 2>/dev/null | cut -f1)
            echo "$worktree: $SIZE"
        fi
    done
fi
```