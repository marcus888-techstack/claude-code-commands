List branches with detailed information

```bash
# Filter pattern from arguments (optional)
FILTER=$ARGUMENTS

echo "=== Git Branches ==="
echo ""

# Current branch
echo "Current branch: $(git branch --show-current)"
echo ""

# Local branches
echo "Local branches:"
if [ -z "$FILTER" ]; then
    git branch -vv
else
    git branch -vv | grep "$FILTER"
fi

echo ""
echo "Remote branches:"
if [ -z "$FILTER" ]; then
    git branch -r -v
else
    git branch -r -v | grep "$FILTER"
fi

echo ""
echo "Branch summary:"
echo "- Total local: $(git branch | wc -l)"
echo "- Total remote: $(git branch -r | wc -l)"
echo "- Feature branches: $(git branch | grep -c "feature/" || echo 0)"
echo "- Release branches: $(git branch | grep -c "release/" || echo 0)"
echo "- Hotfix branches: $(git branch | grep -c "hotfix/" || echo 0)"
```