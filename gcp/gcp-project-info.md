# GCP Project Info

## Purpose
Display comprehensive information about the current or specified Google Cloud project

## Context
Use to get detailed project information including billing, APIs, quotas, and settings. Helpful for troubleshooting, auditing, or understanding project configuration before making changes.

## Parameters
- `$PROJECT_ID` - Optional project ID to inspect
  - Optional
  - Default: Current project
  - Example: `my-project-123`

## Steps

### 1. Determine target project
```bash
if [ -z "$PROJECT_ID" ]; then
    PROJECT_ID=$(gcloud config get-value project 2>/dev/null)
    if [ -z "$PROJECT_ID" ]; then
        echo "Error: No project specified and no default project set"
        exit 1
    fi
fi
echo "Project Information for: $PROJECT_ID"
echo "========================================"
```
Sets the project to inspect, using current if not specified.

### 2. Display basic project details
```bash
gcloud projects describe $PROJECT_ID
```
Shows core project information including name, number, labels, and state.

### 3. Show billing information
```bash
echo -e "\n=== Billing Information ==="
BILLING_ACCOUNT=$(gcloud beta billing projects describe $PROJECT_ID --format="value(billingAccountName)" 2>/dev/null || echo "Not configured")
echo "Billing Account: ${BILLING_ACCOUNT##*/}"
echo "Billing Enabled: $([ "$BILLING_ACCOUNT" != "Not configured" ] && echo "Yes" || echo "No")"
```
Displays billing account linkage and status.

### 4. List enabled APIs
```bash
echo -e "\n=== Enabled APIs (Top 10) ==="
gcloud services list --enabled --project=$PROJECT_ID --format="table(config.name)" --limit=10
API_COUNT=$(gcloud services list --enabled --project=$PROJECT_ID --format="value(config.name)" | wc -l)
echo "Total enabled APIs: $API_COUNT"
```
Shows which Google Cloud APIs are enabled for the project.

### 5. Display IAM policy summary
```bash
echo -e "\n=== IAM Members Summary ==="
gcloud projects get-iam-policy $PROJECT_ID --flatten="bindings[].members" --format="value(bindings.members)" | \
    cut -d: -f1 | sort | uniq -c | sort -rn
```
Provides a count of IAM members by type (user, serviceAccount, group).

### 6. Show project metadata
```bash
echo -e "\n=== Project Metadata ==="
echo "Project Number: $(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')"
echo "Create Time: $(gcloud projects describe $PROJECT_ID --format='value(createTime)')"
echo "Lifecycle State: $(gcloud projects describe $PROJECT_ID --format='value(lifecycleState)')"
```
Displays additional project metadata.

### 7. Check default compute region/zone
```bash
echo -e "\n=== Default Compute Settings ==="
echo "Default Region: $(gcloud config get-value compute/region 2>/dev/null || echo 'Not set')"
echo "Default Zone: $(gcloud config get-value compute/zone 2>/dev/null || echo 'Not set')"
```
Shows default compute settings for the project context.

## Validation
- Project information is displayed without errors
- Billing status is clearly shown
- API count is reasonable (typically 5-50)
- IAM summary shows at least one member

## Error Handling
- **"Project not found"** - Verify project ID and access permissions
- **"Permission denied"** - Need viewer or higher role on the project
- **"API not enabled"** - Some information requires specific APIs to be enabled

## Safety Notes
- Read-only operation, makes no changes
- Some fields require specific permissions to view
- Billing information requires billing.viewer role

## Examples
- **Show current project info**
  ```
  gcp-project-info
  ```
  Displays information about the currently active project

- **Show specific project info**
  ```
  gcp-project-info my-staging-project
  ```
  Displays information about my-staging-project

- **Quick project check**
  ```
  gcp-project-info prod-project-123
  ```
  Reviews production project configuration