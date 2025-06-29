# GCP Secrets Versions List

## Purpose
List all versions of a secret in Google Cloud Secret Manager with their status and metadata

## Context
Use to view version history of a secret, check which versions are enabled or disabled, and track when changes were made. Essential for audit trails and understanding secret rotation history.

## Parameters
- `$SECRET_NAME` - Name of the secret to list versions for
  - Required
  - Example: `database-password`
- `$ARGUMENTS` - Optional filters or format options
  - Optional
  - Example: `--limit=10`, `--filter=state:ENABLED`

## Steps

### 1. Verify secret exists
```bash
if ! gcloud secrets describe $SECRET_NAME &>/dev/null; then
    echo "Error: Secret '$SECRET_NAME' does not exist"
    echo "Run 'gcp-secrets-list' to see available secrets"
    exit 1
fi
```
Ensures the secret exists before listing versions.

### 2. Display secret information
```bash
echo "Secret: $SECRET_NAME"
echo "Project: $(gcloud config get-value project)"
echo "========================================"
```
Shows context about which secret's versions are being listed.

### 3. List all versions
```bash
gcloud secrets versions list $SECRET_NAME $ARGUMENTS
```
Displays all versions with their state, creation time, and other metadata.

### 4. Show version statistics
```bash
echo "========================================"
TOTAL=$(gcloud secrets versions list $SECRET_NAME --format="value(name)" | wc -l)
ENABLED=$(gcloud secrets versions list $SECRET_NAME --filter="state:ENABLED" --format="value(name)" | wc -l)
DISABLED=$(gcloud secrets versions list $SECRET_NAME --filter="state:DISABLED" --format="value(name)" | wc -l)
DESTROYED=$(gcloud secrets versions list $SECRET_NAME --filter="state:DESTROYED" --format="value(name)" | wc -l)

echo "Total versions: $TOTAL"
echo "Enabled: $ENABLED"
echo "Disabled: $DISABLED"
echo "Destroyed: $DESTROYED"
```
Provides summary statistics about version states.

### 5. Show latest version details
```bash
echo -e "\n=== Latest Version ==="
LATEST=$(gcloud secrets versions list $SECRET_NAME --limit=1 --format="value(name)")
if [ -n "$LATEST" ]; then
    gcloud secrets versions describe $LATEST --secret=$SECRET_NAME
else
    echo "No versions found"
fi
```
Displays detailed information about the most recent version.

### 6. Display version aliases
```bash
echo -e "\n=== Version Aliases ==="
echo "latest -> version $(gcloud secrets versions describe latest --secret=$SECRET_NAME --format='value(name)' 2>/dev/null || echo 'N/A')"
```
Shows which actual version the "latest" alias points to.

### 7. Check for old enabled versions
```bash
echo -e "\n=== Rotation Check ==="
ENABLED_COUNT=$(gcloud secrets versions list $SECRET_NAME --filter="state:ENABLED" --format="value(name)" | wc -l)
if [ "$ENABLED_COUNT" -gt 1 ]; then
    echo "Warning: $ENABLED_COUNT versions are enabled. Consider disabling old versions."
    echo "Enabled versions:"
    gcloud secrets versions list $SECRET_NAME --filter="state:ENABLED" --format="table(name,createTime)"
else
    echo "✓ Only one version is enabled (good practice)"
fi
```
Alerts if multiple versions are enabled, which might be a security concern.

## Validation
- Versions are listed with correct states
- Version numbers are sequential
- At least one version exists (for existing secrets)
- Creation times are in chronological order

## Error Handling
- **"Secret not found"** - Check secret name and project
- **"Permission denied"** - Need secretmanager.versions.list permission
- **"No versions"** - Secret exists but has no versions added yet

## Safety Notes
- This only shows metadata, not actual secret values
- Read-only operation, safe to run frequently
- Version numbers are immutable and sequential

## Examples
- **List all versions**
  ```
  gcp-secrets-versions-list api-key
  ```
  Shows all versions of the api-key secret

- **List only enabled versions**
  ```
  gcp-secrets-versions-list database-password "--filter=state:ENABLED"
  ```
  Shows only currently enabled versions

- **List recent versions**
  ```
  gcp-secrets-versions-list ssl-cert "--limit=5"
  ```
  Shows the 5 most recent versions

- **List versions created this month**
  ```
  gcp-secrets-versions-list auth-token "--filter=createTime>2024-01-01"
  ```
  Shows versions created after a specific date