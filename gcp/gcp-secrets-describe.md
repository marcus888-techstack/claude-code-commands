# GCP Secrets Describe

## Purpose
Display detailed metadata and configuration for a specific secret in Google Cloud Secret Manager

## Context
Use to inspect a secret's configuration, replication settings, labels, creation time, and IAM bindings. Useful for understanding how a secret is configured without accessing its actual value.

## Parameters
- `$SECRET_NAME` - Name of the secret to describe
  - Required
  - Example: `database-password`

## Steps

### 1. Verify secret exists
```bash
if ! gcloud secrets describe $SECRET_NAME &>/dev/null; then
    echo "Error: Secret '$SECRET_NAME' does not exist"
    echo "Run 'gcp-secrets-list' to see available secrets"
    exit 1
fi
```
Ensures the secret exists before displaying details.

### 2. Display basic information
```bash
echo "=== Secret Details ==="
gcloud secrets describe $SECRET_NAME
```
Shows core secret metadata including name, creation time, and state.

### 3. Show replication configuration
```bash
echo -e "\n=== Replication Policy ==="
gcloud secrets describe $SECRET_NAME --format="yaml(replication)"
```
Displays how the secret is replicated across regions.

### 4. Display labels
```bash
echo -e "\n=== Labels ==="
LABELS=$(gcloud secrets describe $SECRET_NAME --format="value(labels)")
if [ -z "$LABELS" ]; then
    echo "No labels attached"
else
    gcloud secrets describe $SECRET_NAME --format="table(labels.list())"
fi
```
Shows any labels used for organization or automation.

### 5. Show version information
```bash
echo -e "\n=== Version Summary ==="
TOTAL_VERSIONS=$(gcloud secrets versions list $SECRET_NAME --format="value(name)" | wc -l)
ENABLED_VERSIONS=$(gcloud secrets versions list $SECRET_NAME --filter="state:ENABLED" --format="value(name)" | wc -l)
LATEST_VERSION=$(gcloud secrets versions list $SECRET_NAME --limit=1 --format="value(name)")

echo "Total versions: $TOTAL_VERSIONS"
echo "Enabled versions: $ENABLED_VERSIONS"
echo "Latest version: ${LATEST_VERSION:-none}"

if [ "$TOTAL_VERSIONS" -gt 0 ]; then
    echo -e "\nLatest version details:"
    gcloud secrets versions describe $LATEST_VERSION --secret=$SECRET_NAME --format="table(name,state,createTime)"
fi
```
Provides version statistics and latest version details.

### 6. Display IAM policy
```bash
echo -e "\n=== IAM Policy ==="
gcloud secrets get-iam-policy $SECRET_NAME --format="yaml(bindings)" || echo "No IAM bindings found"
```
Shows who has access to the secret and their roles.

### 7. Check secret usage
```bash
echo -e "\n=== Usage Information ==="
echo "Resource name: projects/$(gcloud config get-value project)/secrets/$SECRET_NAME"
echo "Latest version path: projects/$(gcloud config get-value project)/secrets/$SECRET_NAME/versions/latest"
echo ""
echo "Access in applications:"
echo "- GCP Console: https://console.cloud.google.com/security/secret-manager/secret/$SECRET_NAME"
echo "- API: projects/PROJECT_ID/secrets/$SECRET_NAME/versions/latest"
echo "- gcloud: gcloud secrets versions access latest --secret=$SECRET_NAME"
```
Provides information on how to use the secret in various contexts.

### 8. Show audit configuration
```bash
echo -e "\n=== Audit Configuration ==="
if gcloud logging read "resource.type=secretmanager.googleapis.com/Secret AND resource.labels.secret_id=$SECRET_NAME" --limit=1 --format="value(timestamp)" &>/dev/null; then
    echo "✓ Audit logging is configured"
    LAST_ACCESS=$(gcloud logging read "resource.type=secretmanager.googleapis.com/Secret AND resource.labels.secret_id=$SECRET_NAME" --limit=1 --format="value(timestamp)" 2>/dev/null)
    echo "Last logged access: ${LAST_ACCESS:-Unknown}"
else
    echo "No audit logs found for this secret"
fi
```
Checks if audit logging is capturing access to this secret.

## Validation
- All sections display without errors
- Secret metadata is complete
- Version counts are consistent
- IAM policy is readable

## Error Handling
- **"Secret not found"** - Check secret name and current project
- **"Permission denied"** - Need secretmanager.secrets.get permission
- **"Invalid format"** - Update gcloud SDK if format options fail

## Safety Notes
- This command only shows metadata, never actual secret values
- Read-only operation, safe to run frequently
- IAM policy display requires additional permissions

## Examples
- **Describe a secret**
  ```
  gcp-secrets-describe api-key-production
  ```
  Shows full details about the api-key-production secret

- **Quick secret check**
  ```
  gcp-secrets-describe database-password
  ```
  Verifies secret configuration and access

- **Investigate secret issues**
  ```
  gcp-secrets-describe problematic-secret
  ```
  Helps troubleshoot secret access or configuration problems