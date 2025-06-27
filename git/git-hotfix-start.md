Start a hotfix branch from master for urgent production fixes

```bash
git checkout master
git pull origin master
git checkout -b hotfix/$ARGUMENTS
git push -u origin hotfix/$ARGUMENTS
```