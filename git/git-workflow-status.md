Show comprehensive workflow status including branches, versions, and pending work

```bash
echo "=== Git Workflow Status ==="
echo ""

# Current branch
echo "Current Branch:"
git branch --show-current
echo ""

# Latest versions
echo "Latest Versions:"
echo "Main: $(git describe --tags --abbrev=0 main 2>/dev/null || echo 'No tags')"
echo "Develop: $(git describe --tags --abbrev=0 develop 2>/dev/null || echo 'No tags')"
echo ""

# Active branches
echo "Active Branches:"
git branch -a | grep -E "(feature/|release/|hotfix/)" | sed 's/remotes\/origin\///'
echo ""

# Uncommitted changes
echo "Working Directory:"
git status -s || echo "✓ Clean"
echo ""

# Pending merges
echo "Branches ready to merge:"
git branch --no-merged develop | grep -E "(feature/)" || echo "None"
echo ""

# Recent commits
echo "Recent Activity:"
git log --oneline --graph --decorate -10
```