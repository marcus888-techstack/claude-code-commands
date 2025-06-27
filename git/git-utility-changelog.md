Generate changelog from git tags and commits

```bash
# Parse version range from arguments
RANGE=$ARGUMENTS

echo "# Changelog"
echo ""
echo "Generated on $(date +%Y-%m-%d)"
echo ""

if [ -z "$RANGE" ]; then
    # Generate full changelog from all tags
    TAGS=$(git tag -l --sort=-version:refname | grep -E "^v[0-9]+\.[0-9]+\.[0-9]+$")
    
    PREV_TAG=""
    for TAG in $TAGS; do
        echo "## $TAG"
        
        # Get tag date
        TAG_DATE=$(git log -1 --format=%ai $TAG | cut -d' ' -f1)
        echo "_Released: $TAG_DATE_"
        echo ""
        
        # Get commits since previous tag
        if [ -z "$PREV_TAG" ]; then
            # First tag - show all commits
            git log $TAG --pretty=format:"- %s" --no-merges
        else
            # Show commits between tags
            git log $TAG...$PREV_TAG --pretty=format:"- %s" --no-merges
        fi
        
        echo ""
        echo ""
        PREV_TAG=$TAG
    done
else
    # Generate changelog for specific range
    echo "## Changes from $RANGE"
    echo ""
    
    # Group by type
    echo "### Features"
    git log $RANGE --grep="^feat" --pretty=format:"- %s" --no-merges
    
    echo ""
    echo "### Bug Fixes"
    git log $RANGE --grep="^fix" --pretty=format:"- %s" --no-merges
    
    echo ""
    echo "### Other Changes"
    git log $RANGE --pretty=format:"- %s" --no-merges | grep -v "^- feat" | grep -v "^- fix"
fi
```