# GCP Secrets Delete

## Purpose
Delete a secret and all its versions from Google Cloud Secret Manager

## Context
Use when a secret is no longer needed and should be permanently removed. This is a destructive operation that cannot be undone. All versions of the secret will be deleted. Consider disabling versions instead if you might need the secret later.

## Parameters
- `$SECRET_NAME` - Name of the secret to delete
  - Required
  - Example: `old-api-key`
- `$FORCE` - Skip confirmation prompt
  - Optional
  - Example: `--force`

## Steps

### 1. Verify secret exists
```bash
if ! gcloud secrets describe $SECRET_NAME &>/dev/null; then
    echo "Error: Secret '$SECRET_NAME' does not exist"
    exit 1
fi
```
Ensures the secret exists before attempting deletion.

### 2. Display secret information
```bash
echo "=== Secret to Delete ==="
gcloud secrets describe $SECRET_NAME --format="table(name,createTime,replication.automatic)"
```
Shows details about the secret that will be deleted.

### 3. Show version count
```bash
VERSION_COUNT=$(gcloud secrets versions list $SECRET_NAME --format="value(name)" | wc -l)
echo -e "\nTotal versions: $VERSION_COUNT"
ENABLED_COUNT=$(gcloud secrets versions list $SECRET_NAME --filter="state:ENABLED" --format="value(name)" | wc -l)
echo "Enabled versions: $ENABLED_COUNT"
```
Displays how many versions will be deleted.

### 4. Check for recent access
```bash
echo -e "\n=== Recent Activity ==="
LATEST_VERSION=$(gcloud secrets versions list $SECRET_NAME --limit=1 --format="value(name,createTime)")
echo "Latest version: $LATEST_VERSION"
```
Shows recent activity to help confirm deletion decision.

### 5. List dependent resources (if possible)
```bash
echo -e "\n=== Checking Dependencies ==="
echo "IAM bindings:"
gcloud secrets get-iam-policy $SECRET_NAME --format="table(bindings.members.flatten())" 2>/dev/null | head -10 || echo "None found"
```
Attempts to identify resources that might be using this secret.

### 6. Confirm deletion
```bash
if [ "$FORCE" != "--force" ]; then
    echo -e "\n⚠️  WARNING: This will permanently delete the secret and ALL its versions!"
    echo "Secret name: $SECRET_NAME"
    read -p "Type the secret name to confirm deletion: " CONFIRM_NAME
    
    if [ "$CONFIRM_NAME" != "$SECRET_NAME" ]; then
        echo "Confirmation failed. Secret not deleted."
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

### 7. Delete the secret
```bash
echo -e "\nDeleting secret..."
gcloud secrets delete $SECRET_NAME --quiet
```
Performs the actual deletion. The --quiet flag suppresses additional prompts.

### 8. Verify deletion
```bash
if gcloud secrets describe $SECRET_NAME &>/dev/null; then
    echo "Error: Secret still exists. Deletion may have failed."
    exit 1
else
    echo "✓ Secret '$SECRET_NAME' has been deleted successfully"
fi
```
Confirms the secret no longer exists.

### 9. Log deletion
```bash
echo -e "\n=== Deletion Record ==="
echo "Deleted by: $(gcloud config get-value account)"
echo "Deleted at: $(date)"
echo "Project: $(gcloud config get-value project)"
echo "Consider documenting this deletion in your team's records"
```
Records deletion details for audit purposes.

## Validation
- Secret no longer exists after deletion
- Cannot access any versions
- Secret name is freed for reuse

## Error Handling
- **"Secret not found"** - Secret doesn't exist or already deleted
- **"Permission denied"** - Need secretmanager.secrets.delete permission
- **"Resource in use"** - Check if applications are still using the secret

## Safety Notes
- This operation is IRREVERSIBLE
- All versions are permanently deleted
- Consider exporting important values before deletion
- Update all applications using this secret first
- Consider disabling instead of deleting if unsure

## Examples
- **Delete with confirmation**
  ```
  gcp-secrets-delete old-api-key
  ```
  Deletes secret after confirmation prompts

- **Force delete (scripts)**
  ```
  gcp-secrets-delete temp-secret --force
  ```
  Deletes without confirmation (use carefully)

- **Delete after backup**
  ```
  gcloud secrets versions access latest --secret=old-cert > backup.pem
  gcp-secrets-delete old-cert
  ```
  Backs up latest version before deletion