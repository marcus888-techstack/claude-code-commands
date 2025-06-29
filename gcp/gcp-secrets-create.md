# GCP Secrets Create

## Purpose
Create a new secret in Google Cloud Secret Manager with an initial value

## Context
Use when you need to store sensitive data like API keys, passwords, or certificates securely. Secrets are encrypted at rest and can be accessed programmatically by applications with proper permissions.

## Parameters
- `$SECRET_NAME` - Name for the new secret
  - Required
  - Example: `api-key-production`
- `$SECRET_VALUE` - Initial value for the secret (optional, can add later)
  - Optional
  - Example: `my-secret-value`

## Steps

### 1. Validate secret name
```bash
if [[ ! "$SECRET_NAME" =~ ^[a-zA-Z0-9_-]+$ ]]; then
    echo "Error: Secret name must contain only letters, numbers, hyphens, and underscores"
    exit 1
fi
```
Ensures the secret name follows GCP naming conventions.

### 2. Check if secret already exists
```bash
if gcloud secrets describe $SECRET_NAME &>/dev/null; then
    echo "Error: Secret '$SECRET_NAME' already exists"
    echo "Use 'gcp-secrets-versions-add' to add a new version"
    exit 1
fi
```
Prevents accidental overwriting of existing secrets.

### 3. Create the secret
```bash
gcloud secrets create $SECRET_NAME --replication-policy="automatic"
```
Creates the secret with automatic replication for high availability.

### 4. Add initial value if provided
```bash
if [ -n "$SECRET_VALUE" ]; then
    echo -n "$SECRET_VALUE" | gcloud secrets versions add $SECRET_NAME --data-file=-
    echo "Added initial version to secret"
else
    echo "Secret created without initial value"
    echo "Add a value with: gcp-secrets-versions-add $SECRET_NAME"
fi
```
Optionally adds the first version of the secret value.

### 5. Set common labels
```bash
echo "Adding metadata labels..."
gcloud secrets update $SECRET_NAME --update-labels=created-by=$(whoami),created-date=$(date +%Y-%m-%d)
```
Adds tracking labels to the secret.

### 6. Display secret information
```bash
echo -e "\n=== Secret Created Successfully ==="
gcloud secrets describe $SECRET_NAME --format="table(name,createTime,replication.automatic.customerManagedEncryption)"
```
Shows the newly created secret's details.

### 7. Show access instructions
```bash
echo -e "\n=== Next Steps ==="
echo "1. Grant access: gcloud secrets add-iam-policy-binding $SECRET_NAME --member=USER_OR_SERVICE_ACCOUNT --role=roles/secretmanager.secretAccessor"
echo "2. Access value: gcloud secrets versions access latest --secret=$SECRET_NAME"
echo "3. Use in app: projects/PROJECT_ID/secrets/$SECRET_NAME/versions/latest"
```
Provides guidance on using the new secret.

## Validation
- Secret is created successfully
- Can describe the secret
- If value provided, version 1 exists
- Labels are applied correctly

## Error Handling
- **"already exists"** - Secret name is taken, choose a different name
- **"Permission denied"** - Need secretmanager.secrets.create permission
- **"Invalid secret name"** - Name must be 1-255 characters, alphanumeric, hyphens, underscores
- **"API not enabled"** - Enable Secret Manager API first

## Safety Notes
- Secret values are sensitive - avoid logging or displaying them
- Use --data-file instead of command line for values to avoid shell history
- Consider using service accounts for application access
- Enable audit logging for secret access

## Examples
- **Create empty secret**
  ```
  gcp-secrets-create database-password
  ```
  Creates secret without initial value

- **Create secret with value**
  ```
  gcp-secrets-create api-key-stripe "sk_test_123456"
  ```
  Creates secret with initial value

- **Create from file**
  ```
  cat private-key.pem | gcp-secrets-create ssl-private-key
  ```
  Creates secret from file contents