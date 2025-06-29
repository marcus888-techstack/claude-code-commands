# GCP Project List

## Purpose
List all accessible Google Cloud projects with detailed information

## Context
Use to discover available projects in your organization or personal account. Shows project IDs, names, and metadata. Essential for finding the right project before switching or deploying resources.

## Parameters
- `$ARGUMENTS` - Optional filters or format options
  - Optional
  - Example: `--limit=10`, `--filter=lifecycleState:ACTIVE`

## Steps

### 1. Display current project context
```bash
echo "Current project: $(gcloud config get-value project 2>/dev/null || echo 'None set')"
echo "---"
```
Shows which project is currently active for context.

### 2. List all accessible projects
```bash
gcloud projects list $ARGUMENTS
```
Displays projects with ID, name, and project number. Default shows all accessible projects.

### 3. Count total projects
```bash
PROJECT_COUNT=$(gcloud projects list --format="value(projectId)" | wc -l)
echo "---"
echo "Total accessible projects: $PROJECT_COUNT"
```
Provides a summary count of accessible projects.

### 4. Show projects by lifecycle state
```bash
echo "Active projects: $(gcloud projects list --filter=lifecycleState:ACTIVE --format="value(projectId)" | wc -l)"
echo "Delete requested: $(gcloud projects list --filter=lifecycleState:DELETE_REQUESTED --format="value(projectId)" | wc -l)"
```
Breaks down projects by their current state.

### 5. List projects with labels (if any)
```bash
echo "---"
echo "Projects with labels:"
gcloud projects list --format="table(projectId,labels.list())" --filter="labels:*"
```
Shows projects that have labels applied for organization.

## Validation
- At least one project is listed (unless no access)
- Project IDs are in correct format (lowercase, hyphens)
- Current project is identified if set

## Error Handling
- **"Listed 0 items"** - No projects accessible with current account
- **"Permission denied"** - Account lacks project listing permissions
- **"Request had invalid authentication"** - Need to re-authenticate
- **"Quota exceeded"** - Too many API requests, wait before retrying

## Safety Notes
- Read-only operation, safe to run frequently
- May take time with many projects
- Requires resourcemanager.projects.list permission

## Examples
- **List all projects**
  ```
  gcp-project-list
  ```
  Shows all accessible projects with full details

- **List first 5 projects**
  ```
  gcp-project-list "--limit=5"
  ```
  Shows only the first 5 projects

- **List only active projects**
  ```
  gcp-project-list "--filter=lifecycleState:ACTIVE"
  ```
  Excludes projects marked for deletion

- **List projects with specific label**
  ```
  gcp-project-list "--filter=labels.env:production"
  ```
  Shows only projects labeled with env=production