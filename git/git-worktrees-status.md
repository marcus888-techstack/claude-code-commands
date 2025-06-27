Show detailed status of all worktrees

```bash
# Optional filter from arguments
FILTER=$ARGUMENTS

echo "=== Git Worktrees Status ==="
echo ""

# Get current directory to highlight current worktree
CURRENT_DIR=$(pwd)

# Process each worktree
git worktree list --porcelain | awk -v filter="$FILTER" -v current="$CURRENT_DIR" '
    /^worktree/ {
        path=$2
        is_current = (path == current) ? " [CURRENT]" : ""
    }
    /^HEAD/ {head=substr($2, 1, 7)}
    /^branch/ {
        branch=$2
        gsub("refs/heads/", "", branch)
    }
    /^detached/ {branch="(detached)"}
    /^$/ {
        if (filter == "" || branch ~ filter || path ~ filter) {
            print "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
            print "Worktree: " path is_current
            print "Branch:   " branch
            print "HEAD:     " head
            
            # Get additional info using system calls
            cmd = "cd \"" path "\" 2>/dev/null && git status --porcelain | wc -l"
            cmd | getline changes
            close(cmd)
            
            if (changes > 0) {
                print "Status:   " changes " uncommitted changes"
            } else {
                print "Status:   Clean"
            }
            
            # Check if merged
            cmd = "git branch --merged develop 2>/dev/null | grep -c \"^  " branch "$\""
            cmd | getline merged_dev
            close(cmd)
            
            cmd = "git branch --merged main 2>/dev/null | grep -c \"^  " branch "$\""
            cmd | getline merged_main
            close(cmd)
            
            if (merged_dev > 0 || merged_main > 0) {
                print "Merged:   Yes ✓"
            } else if (branch != "main" && branch != "develop") {
                print "Merged:   No"
            }
            
            print ""
        }
        path=""; head=""; branch=""; changes=0
    }
'

# Summary statistics
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "Summary:"
echo ""

TOTAL=$(git worktree list | wc -l)
FEATURE_COUNT=$(git worktree list | grep -c "/feature/" || echo 0)
HOTFIX_COUNT=$(git worktree list | grep -c "/hotfix/" || echo 0)

echo "Total worktrees:  $TOTAL"
echo "Features:         $FEATURE_COUNT"
echo "Hotfixes:         $HOTFIX_COUNT"
echo "Main/Develop:     $((TOTAL - FEATURE_COUNT - HOTFIX_COUNT))"

# Check for issues
echo ""
STALE=$(git worktree list --porcelain | grep -c "prunable" || echo 0)
if [ $STALE -gt 0 ]; then
    echo "⚠️  Found $STALE stale worktree(s)"
    echo "   Run: git worktree prune"
fi

# Disk usage
if command -v du &> /dev/null; then
    echo ""
    echo "Disk usage:"
    TOTAL_SIZE=$(git worktree list --porcelain | grep "^worktree" | cut -d' ' -f2 | xargs du -shc 2>/dev/null | tail -1 | cut -f1)
    echo "Total: $TOTAL_SIZE"
fi
```