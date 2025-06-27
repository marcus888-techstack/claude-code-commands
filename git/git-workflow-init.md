Initialize Git workflow with main and develop branches (simplified for tag-based releases)

```bash
# Ensure we're on main branch
git checkout main
git pull origin main

# Create develop branch if it doesn't exist
if ! git show-ref --verify --quiet refs/heads/develop; then
    git checkout -b develop
    git push -u origin develop
    echo "✓ Created develop branch"
else
    echo "✓ Develop branch already exists"
fi

# Set branch protection
git config --global branch.main.protect true
git config --global branch.develop.protect true

# Create initial version tag if no tags exist
if [ -z "$(git tag -l)" ]; then
    git tag -a "v0.1.0" -m "Initial release"
    git push origin v0.1.0
    
    # Tag develop branch
    git checkout develop
    git tag -a "v0.1.0_dev" -m "Development starts from v0.1.0"
    git push origin v0.1.0_dev
    
    echo "✓ Created initial version: v0.1.0"
else
    echo "✓ Tags already exist"
fi

echo ""
echo "=== Git Workflow Initialized ==="
echo "✓ Main branch: Production-ready code"
echo "✓ Develop branch: Integration branch"
echo "✓ Branch protection: Enabled"
echo ""
echo "Workflow:"
echo "1. Features: Branch from develop"
echo "2. Releases: Tag from develop, merge to main"
echo "3. Hotfixes: Branch from main, merge to both"
echo ""
echo "No release branches needed - use tags instead!"
```