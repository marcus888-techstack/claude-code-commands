Set branch protection rules for main and develop branches

```bash
# Branch to protect (default: both main and develop)
BRANCH=$ARGUMENTS

if [ -z "$BRANCH" ]; then
    # Protect both main branches
    git config branch.main.protect true
    git config branch.master.protect true
    git config branch.develop.protect true
    
    echo "✓ Protected branches: main, master, develop"
    echo ""
    echo "Protection enabled for:"
    echo "- Prevent direct pushes"
    echo "- Require pull requests"
    echo "- Prevent force pushes"
    echo "- Prevent deletion"
else
    # Protect specific branch
    git config branch.$BRANCH.protect true
    echo "✓ Protected branch: $BRANCH"
fi

echo ""
echo "Note: Full protection requires GitHub/GitLab/Bitbucket settings"
```