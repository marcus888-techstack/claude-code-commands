Complete hotfix by merging to master and develop

```bash
# Merge to master and tag
git checkout master
git pull origin master
git merge --no-ff hotfix/$ARGUMENTS
git tag -a "hotfix-$ARGUMENTS" -m "Hotfix: $ARGUMENTS"
git push origin master
git push origin "hotfix-$ARGUMENTS"

# Merge to develop
git checkout develop
git pull origin develop
git merge --no-ff hotfix/$ARGUMENTS
git push origin develop

# Cleanup
git branch -d hotfix/$ARGUMENTS
git push origin --delete hotfix/$ARGUMENTS
```