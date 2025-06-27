List version tags in semantic order

```bash
# Filter pattern from arguments (optional)
FILTER=$ARGUMENTS

echo "=== Version Tags ==="
echo ""

if [ -z "$FILTER" ]; then
    # List all versions
    echo "Production versions:"
    git tag -l --sort=-version:refname | grep -E "^v[0-9]+\.[0-9]+\.[0-9]+$" | head -10
    
    echo ""
    echo "Development versions:"
    git tag -l --sort=-version:refname | grep "_dev$" | head -10
    
    echo ""
    echo "Pre-release versions:"
    git tag -l --sort=-version:refname | grep -E "^v[0-9]+\.[0-9]+\.[0-9]+-(alpha|beta|rc)" | head -10
else
    # List filtered versions
    echo "Versions matching '$FILTER':"
    git tag -l --sort=-version:refname | grep "$FILTER"
fi

echo ""
echo "Total tags: $(git tag -l | wc -l)"
```