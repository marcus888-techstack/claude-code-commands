Initialize Git worktree structure for parallel development

```bash
# Parse project name from arguments (optional)
PROJECT_NAME=${ARGUMENTS:-$(basename $(git rev-parse --show-toplevel))}

echo "=== Setting up Git Worktrees for: $PROJECT_NAME ==="
echo ""

# Get repository root
REPO_ROOT=$(git rev-parse --show-toplevel)
PARENT_DIR=$(dirname "$REPO_ROOT")

# Create project structure
echo "Creating worktree structure..."
mkdir -p "$PARENT_DIR/$PROJECT_NAME"/{main,develop,features,hotfixes}

# Move current repository to main (if not already)
if [ "$REPO_ROOT" != "$PARENT_DIR/$PROJECT_NAME/main" ]; then
    echo "Moving repository to main worktree..."
    mv "$REPO_ROOT" "$PARENT_DIR/$PROJECT_NAME/main"
    cd "$PARENT_DIR/$PROJECT_NAME/main"
fi

# Create develop worktree
if ! git worktree list | grep -q "develop"; then
    git worktree add ../develop develop
    echo "✓ Created develop worktree"
else
    echo "✓ Develop worktree already exists"
fi

# Set up directory structure
echo ""
echo "=== Worktree Structure Created ==="
echo "$PROJECT_NAME/"
echo "├── main/          # Production branch"
echo "├── develop/       # Development branch"
echo "├── features/      # Feature branches"
echo "└── hotfixes/      # Hotfix branches"

# Create convenience script
cat > "$PARENT_DIR/$PROJECT_NAME/switch-worktree.sh" << 'EOF'
#!/bin/bash
# Quick switch between worktrees

WORKTREE=$1
if [ -z "$WORKTREE" ]; then
    echo "Available worktrees:"
    ls -d */ 2>/dev/null | grep -v features/ | grep -v hotfixes/
    ls -d features/*/ 2>/dev/null 2>/dev/null
    ls -d hotfixes/*/ 2>/dev/null 2>/dev/null
    exit 1
fi

if [ -d "$WORKTREE" ]; then
    cd "$WORKTREE"
    exec $SHELL
else
    echo "Worktree not found: $WORKTREE"
fi
EOF

chmod +x "$PARENT_DIR/$PROJECT_NAME/switch-worktree.sh"

echo ""
echo "✓ Setup complete!"
echo ""
echo "Quick commands:"
echo "- Switch to develop: cd ../develop"
echo "- Create feature: git worktree add ../features/my-feature -b feature/my-feature"
echo "- List worktrees: git worktree list"
echo "- Use switch script: ../switch-worktree.sh develop"
```