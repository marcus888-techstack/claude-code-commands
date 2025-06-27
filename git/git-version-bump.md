Bump version number in project files

```bash
# Parse version type or specific version from arguments
VERSION_ARG=$ARGUMENTS

if [ -z "$VERSION_ARG" ]; then
    echo "Usage: /git:version:bump <version|major|minor|patch>"
    echo ""
    echo "Examples:"
    echo "  /git:version:bump patch     # 1.0.0 -> 1.0.1"
    echo "  /git:version:bump minor     # 1.0.0 -> 1.1.0"
    echo "  /git:version:bump major     # 1.0.0 -> 2.0.0"
    echo "  /git:version:bump 1.2.3     # Set specific version"
    exit 1
fi

# Check if package.json exists
if [ -f "package.json" ]; then
    echo "Updating version in package.json..."
    
    # Use npm version command
    npm version $VERSION_ARG --no-git-tag-version
    NEW_VERSION=$(node -p "require('./package.json').version")
    
    # Stage changes
    git add package.json
    [ -f "package-lock.json" ] && git add package-lock.json
    
    # Commit
    git commit -m "chore: bump version to v$NEW_VERSION"
    git push origin $(git branch --show-current)
    
    echo "✓ Version bumped to v$NEW_VERSION"
    echo ""
    echo "Next steps:"
    echo "1. Complete any remaining changes"
    echo "2. Create release: /git:release:tag v$NEW_VERSION"
    
elif [ -f "Cargo.toml" ]; then
    echo "Updating version in Cargo.toml..."
    # Handle Rust projects
    cargo set-version $VERSION_ARG
    git add Cargo.toml Cargo.lock
    git commit -m "chore: bump version to $VERSION_ARG"
    git push origin $(git branch --show-current)
    
elif [ -f "pyproject.toml" ] || [ -f "setup.py" ]; then
    echo "Python project detected"
    echo "Please update version manually in pyproject.toml or setup.py"
    echo "Then commit with: git commit -m \"chore: bump version to $VERSION_ARG\""
    
else
    echo "No recognized project file found (package.json, Cargo.toml, etc.)"
    echo "Please update version manually in your project files"
fi
```