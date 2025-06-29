# GCP Project Switch

## Purpose
Switch the active Google Cloud project for all subsequent gcloud commands

## Context
Use when you need to work with resources in a different project. Changes the default project context for all gcloud operations. Essential when managing resources across multiple projects.

## Parameters
- `$PROJECT_ID` - The project ID to switch to
  - Required
  - Example: `my-project-123`

## Steps

### 1. Show current project
```bash
CURRENT_PROJECT=$(gcloud config get-value project 2>/dev/null || echo "None")
echo "Current project: $CURRENT_PROJECT"
```
Records the current project before switching.

### 2. Verify target project exists
```bash
if ! gcloud projects describe $PROJECT_ID &>/dev/null; then
    echo "Error: Project '$PROJECT_ID' not found or not accessible"
    echo "Run 'gcp-project-list' to see available projects"
    exit 1
fi
```
Ensures the target project exists and is accessible.

### 3. Switch to the new project
```bash
gcloud config set project $PROJECT_ID
```
Updates the active project configuration.

### 4. Verify the switch
```bash
echo "Switched to project: $(gcloud config get-value project)"
```
Confirms the project has been changed.

### 5. Display project details
```bash
echo "---"
gcloud projects describe $PROJECT_ID --format="table(projectId,name,projectNumber,lifecycleState)"
```
Shows key information about the newly active project.

### 6. Update application default project
```bash
export GOOGLE_CLOUD_PROJECT=$PROJECT_ID
echo "Updated GOOGLE_CLOUD_PROJECT environment variable"
```
Ensures environment variables reflect the new project.

### 7. Test project access
```bash
echo "---"
echo "Testing access..."
gcloud compute regions list --limit=1 &>/dev/null && echo "✓ Compute API accessible" || echo "✗ Compute API not enabled or accessible"
```
Quick test to verify API access in the new project.

## Validation
- Project ID matches requested project
- Can describe the project successfully
- Subsequent commands use the new project context

## Error Handling
- **"Project not found"** - Check project ID spelling and access permissions
- **"Permission denied"** - Ensure your account has access to the project
- **"Invalid project ID"** - Project IDs must be 6-30 characters, lowercase letters, digits, and hyphens

## Safety Notes
- Changes affect all subsequent gcloud commands
- Previous project remains unchanged (not deleted)
- May need to re-authenticate for some project-specific resources

## Examples
- **Switch to development project**
  ```
  gcp-project-switch my-dev-project
  ```
  Sets my-dev-project as the active project

- **Switch to production project**
  ```
  gcp-project-switch company-prod-123
  ```
  Sets company-prod-123 as the active project