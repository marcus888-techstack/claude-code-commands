# Git Release Deploy

## Purpose
Deploy from a tagged release

## Context
Use when deploying a specific version to production. This command helps you checkout a release tag and provides deployment guidance. Supports both traditional checkout and worktree-based deployments.

## Parameters
- `$ARGUMENTS` - Version tag to deploy
  - Required
  - Example: `v1.2.0` or `1.2.0` (v prefix added automatically)

## Steps

### 1. Validate tag input
If no tag provided, show recent release tags for selection.

### 2. Ensure tag format
```bash
if [[ ! "$TAG" =~ ^v ]]; then
    TAG="v$TAG"
fi
```
Adds 'v' prefix if not present.

### 3. Verify tag exists
```bash
git rev-parse "$TAG"
```
Checks that the tag exists in the repository.

### 4. Display tag information
```bash
git show "$TAG" --no-patch
```
Shows tag details including tagger and date.

### 5. Checkout deployment code
Two options:
- **With worktrees**: Creates separate deployment directory
- **Traditional**: Checks out tag in current directory

### 6. Display deployment checklist
Provides reminders for:
- Building application
- Running production tests
- Updating environment variables
- Backing up database
- Notifying team

### 7. Show deployment commands
Examples for common deployment scenarios:
- npm/yarn builds
- Docker image creation
- Kubernetes deployments

## Validation
- Tag exists and is checked out
- Deployment directory is ready
- Checklist helps ensure safe deployment

## Error Handling
- **"Tag not found"** - Shows available tags for selection
- **"Deployment directory already exists"** - Uses existing directory
- **"No recent tags"** - Check if any releases have been tagged

## Safety Notes
- Always backup before deploying
- Test the specific version before production
- Have rollback plan ready
- Document deployment in team chat/logs

## Examples
- **Deploy specific version**
  ```
  git-release-deploy v1.2.0
  ```
  Deploys version 1.2.0

- **Deploy without v prefix**
  ```
  git-release-deploy 2.0.0
  ```
  Automatically adds v prefix and deploys v2.0.0