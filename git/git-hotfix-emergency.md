Fast-track critical hotfix deployment with immediate push to production

```bash
# Create hotfix from main
git checkout main
git pull origin main
git checkout -b hotfix/$ARGUMENTS

echo "⚠️  EMERGENCY HOTFIX MODE"
echo "Branch created: hotfix/$ARGUMENTS"
echo ""
echo "1. Make your critical fixes"
echo "2. Test thoroughly"
echo "3. Run: /git/hotfix/finish $ARGUMENTS"
echo ""
echo "This will:"
echo "- Merge to main with version tag"
echo "- Deploy to production"
echo "- Sync with develop branch"
```