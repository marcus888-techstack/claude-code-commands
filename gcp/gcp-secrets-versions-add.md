# GCP Secrets Versions Add

## Purpose
Add a new version to an existing secret in Google Cloud Secret Manager

## Context
Use when you need to update a secret's value, such as rotating API keys, updating passwords, or refreshing certificates. Each version is immutable and maintains history for audit and rollback purposes.

## Parameters
- `$SECRET_NAME` - Name of the existing secret
  - Required
  - Example: `database-password`
- `$SECRET_VALUE` - New value for the secret
  - Required
  - Example: `new-password-123`

## Steps

### 1. Verify secret exists
```bash
if ! gcloud secrets describe $SECRET_NAME &>/dev/null; then
    echo "Error: Secret '$SECRET_NAME' does not exist"
    echo "Create it first with: gcp-secrets-create $SECRET_NAME"
    exit 1
fi
```
Ensures the target secret exists before adding a version.

### 2. Get current version count
```bash
CURRENT_VERSION=$(gcloud secrets versions list $SECRET_NAME --limit=1 --format="value(name)")
echo "Current latest version: ${CURRENT_VERSION:-none}"
```
Shows the current latest version for reference.

### 3. Add new version
```bash
echo -n "$SECRET_VALUE" | gcloud secrets versions add $SECRET_NAME --data-file=-
```
Creates a new version with the provided value. Using echo -n prevents trailing newline.

### 4. Verify new version
```bash
NEW_VERSION=$(gcloud secrets versions list $SECRET_NAME --limit=1 --format="value(name)")
echo "New version created: $NEW_VERSION"
```
Confirms the new version was created successfully.

### 5. Disable previous versions (optional)
```bash
read -p "Disable previous versions? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    for version in $(gcloud secrets versions list $SECRET_NAME --filter="state:ENABLED" --format="value(name)" | grep -v "^$NEW_VERSION$"); do
        gcloud secrets versions disable $version --secret=$SECRET_NAME
        echo "Disabled version: $version"
    done
fi
```
Optionally disables older versions to prevent their use.

### 6. Show version history
```bash
echo -e "\n=== Version History (last 5) ==="
gcloud secrets versions list $SECRET_NAME --limit=5 --format="table(name,state,createTime)"
```
Displays recent version history for audit purposes.

### 7. Test access to new version
```bash
echo -e "\n=== Verifying Access ==="
if gcloud secrets versions access $NEW_VERSION --secret=$SECRET_NAME >/dev/null 2>&1; then
    echo "✓ New version is accessible"
else
    echo "✗ Error accessing new version"
fi
```
Confirms the new version can be accessed successfully.

## Validation
- New version number is higher than previous
- Version state is ENABLED
- Can access the new version
- Version count increased by 1

## Error Handling
- **"Secret not found"** - Secret must exist before adding versions
- **"Permission denied"** - Need secretmanager.versions.add permission
- **"Invalid value"** - Check for special characters or encoding issues
- **"Quota exceeded"** - Too many versions, consider cleaning up old ones

## Safety Notes
- Old versions remain accessible unless explicitly disabled
- Avoid displaying secret values in logs or output
- Consider impact on running applications before disabling versions
- Use version aliases (like "latest") in applications for automatic updates

## Examples
- **Add new API key version**
  ```
  gcp-secrets-versions-add stripe-api-key "sk_live_newkey123"
  ```
  Updates the Stripe API key secret

- **Update from file**
  ```
  cat new-cert.pem | gcp-secrets-versions-add ssl-certificate
  ```
  Updates certificate from file

- **Rotate database password**
  ```
  gcp-secrets-versions-add db-password "$(openssl rand -base64 32)"
  ```
  Generates and sets a random password