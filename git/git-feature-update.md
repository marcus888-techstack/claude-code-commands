Update feature branch with latest changes from develop

```bash
git checkout develop
git pull origin develop
git checkout feature/$ARGUMENTS
git merge develop
git push origin feature/$ARGUMENTS
```