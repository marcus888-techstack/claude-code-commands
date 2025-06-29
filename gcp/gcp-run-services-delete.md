# GCP Run Services Delete

## Purpose
Delete a Cloud Run service and all its revisions

## Context
Use to remove Cloud Run services that are no longer needed. This permanently deletes the service, all its revisions, and frees up the service name for reuse. The service URL will no longer be accessible.

## Parameters
- `$SERVICE_NAME` - Name of the service to delete
  - Required
  - Example: `my-api`, `old-webapp`
- `$REGION` - Region where the service is deployed
  - Required
  - Example: `us-central1`, `europe-west1`
- `$FORCE` - Skip confirmation prompt
  - Optional
  - Example: `--force`

## Steps

### 1. Verify service exists
```bash
if ! gcloud run services describe $SERVICE_NAME --region=$REGION &>/dev/null; then
    echo "Error: Service '$SERVICE_NAME' not found in region '$REGION'"
    echo "Available services in this region:"
    gcloud run services list --region=$REGION --format="table(name)"
    exit 1
fi
```
Ensures the service exists before attempting deletion.

### 2. Display service information
```bash
echo "=== Service to Delete ==="
gcloud run services describe $SERVICE_NAME --region=$REGION --format="table(
    metadata.name,
    status.url,
    spec.template.spec.containers[0].image,
    metadata.creationTimestamp
)"
```
Shows details about the service to be deleted.

### 3. Check traffic and revisions
```bash
echo -e "\n=== Traffic Distribution ==="
gcloud run services describe $SERVICE_NAME --region=$REGION --format="table(
    spec.traffic[].revisionName,
    spec.traffic[].percent
)"

REVISION_COUNT=$(gcloud run revisions list --service=$SERVICE_NAME --region=$REGION --format="value(name)" | wc -l)
echo -e "\nTotal revisions: $REVISION_COUNT"
```
Shows current traffic allocation and revision count.

### 4. Check for recent activity
```bash
echo -e "\n=== Recent Activity ==="
echo "Last 5 log entries:"
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=$SERVICE_NAME" \
    --limit=5 \
    --format="table(timestamp,jsonPayload.message)" 2>/dev/null | head -10 || echo "No recent logs found"
```
Displays recent activity to help confirm deletion decision.

### 5. Check authentication status
```bash
echo -e "\n=== Access Configuration ==="
if gcloud run services get-iam-policy $SERVICE_NAME --region=$REGION 2>/dev/null | grep -q "allUsers"; then
    echo "⚠️  This is a PUBLIC service (accessible without authentication)"
    SERVICE_URL=$(gcloud run services describe $SERVICE_NAME --region=$REGION --format="value(status.url)")
    echo "Public URL: $SERVICE_URL"
else
    echo "This is a PRIVATE service (requires authentication)"
fi
```
Shows whether the service is publicly accessible.

### 6. Confirm deletion
```bash
if [ "$FORCE" != "--force" ]; then
    echo -e "\n⚠️  WARNING: This will permanently delete the service and all its revisions!"
    echo "Service: $SERVICE_NAME"
    echo "Region: $REGION"
    echo "URL will become inaccessible immediately"
    
    read -p "Type the service name to confirm deletion: " CONFIRM_NAME
    
    if [ "$CONFIRM_NAME" != "$SERVICE_NAME" ]; then
        echo "Confirmation failed. Service not deleted."
        exit 1
    fi
    
    read -p "Are you absolutely sure? (yes/N): " -r
    if [[ ! $REPLY =~ ^yes$ ]]; then
        echo "Deletion cancelled."
        exit 1
    fi
fi
```
Requires explicit confirmation to prevent accidental deletion.

### 7. Delete the service
```bash
echo -e "\nDeleting service..."
gcloud run services delete $SERVICE_NAME --region=$REGION --quiet

if [ $? -eq 0 ]; then
    echo "✓ Service '$SERVICE_NAME' has been deleted successfully"
else
    echo "✗ Failed to delete service"
    exit 1
fi
```
Performs the actual deletion.

### 8. Verify deletion
```bash
echo -e "\n=== Verifying Deletion ==="
if gcloud run services describe $SERVICE_NAME --region=$REGION &>/dev/null; then
    echo "⚠️  Service still exists - deletion may have failed"
    exit 1
else
    echo "✓ Service no longer exists"
fi

# Check if URL is still accessible
if [ -n "$SERVICE_URL" ]; then
    echo -n "Checking URL status: "
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" $SERVICE_URL 2>/dev/null)
    if [ "$HTTP_CODE" = "404" ]; then
        echo "✓ URL returns 404 (as expected)"
    else
        echo "HTTP $HTTP_CODE"
    fi
fi
```
Confirms the service is completely removed.

### 9. Clean up related resources
```bash
echo -e "\n=== Cleanup Suggestions ==="
echo "Consider cleaning up related resources:"
echo ""
echo "1. Container images:"
echo "   - If using Container Registry: gcloud container images list --repository=gcr.io/$PROJECT"
echo "   - If using Artifact Registry: gcloud artifacts docker images list REPOSITORY"
echo ""
echo "2. Service account (if dedicated):"
echo "   gcloud iam service-accounts list --filter='$SERVICE_NAME'"
echo ""
echo "3. Secrets (if used):"
echo "   gcloud secrets list --filter='labels.service:$SERVICE_NAME'"
echo ""
echo "4. Custom domain mappings:"
echo "   gcloud run domain-mappings list --region=$REGION"
```
Suggests cleanup of related resources.

### 10. Log deletion
```bash
echo -e "\n=== Deletion Record ==="
echo "Deleted by: $(gcloud config get-value account)"
echo "Deleted at: $(date)"
echo "Project: $(gcloud config get-value project)"
echo "Service: $SERVICE_NAME"
echo "Region: $REGION"
```
Records deletion details for audit purposes.

## Validation
- Service no longer exists
- URL returns 404
- Service not listed in region
- All revisions are removed

## Error Handling
- **"Service not found"** - Service doesn't exist or wrong region
- **"Permission denied"** - Need run.services.delete permission
- **"Invalid region"** - Check region name is correct
- **"Deletion failed"** - May have dependent resources

## Safety Notes
- This operation is IRREVERSIBLE
- All revisions are permanently deleted
- URL becomes immediately inaccessible
- Service name can be reused after deletion
- No automatic backup of service configuration

## Examples
- **Delete with confirmation**
  ```
  gcp-run-services-delete my-old-api us-central1
  ```
  Deletes service after confirmation prompts

- **Force delete (scripts)**
  ```
  gcp-run-services-delete temp-service us-east1 --force
  ```
  Deletes without confirmation

- **Delete from specific region**
  ```
  gcp-run-services-delete test-app europe-west1
  ```
  Deletes service from European region