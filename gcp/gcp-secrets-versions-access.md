# GCP Secrets Versions Access

## Purpose
Retrieve the value of a specific version of a secret from Google Cloud Secret Manager

## Context
Use to access secret values for local development, debugging, or manual operations. Commonly used to retrieve API keys, passwords, or certificates. Be cautious about where and how you display secret values.

## Parameters
- `$SECRET_NAME` - Name of the secret to access
  - Required
  - Example: `database-password`
- `$VERSION` - Version number or alias to access
  - Optional
  - Default: `latest`
  - Example: `1`, `latest`, `5`

## Steps

### 1. Set default version if not specified
```bash
if [ -z "$VERSION" ]; then
    VERSION="latest"
fi
```
Uses "latest" as the default version for convenience.

### 2. Verify secret exists
```bash
if ! gcloud secrets describe $SECRET_NAME &>/dev/null; then
    echo "Error: Secret '$SECRET_NAME' does not exist"
    exit 1
fi
```
Ensures the secret exists before attempting access.

### 3. Check version exists
```bash
if ! gcloud secrets versions describe $VERSION --secret=$SECRET_NAME &>/dev/null; then
    echo "Error: Version '$VERSION' does not exist for secret '$SECRET_NAME'"
    echo "Available versions:"
    gcloud secrets versions list $SECRET_NAME --limit=5 --format="table(name,state)"
    exit 1
fi
```
Validates the requested version exists and shows alternatives if not.

### 4. Display version metadata
```bash
echo "Accessing secret: $SECRET_NAME"
echo "Version: $VERSION"
VERSION_STATE=$(gcloud secrets versions describe $VERSION --secret=$SECRET_NAME --format="value(state)")
echo "State: $VERSION_STATE"
echo "========================================"
```
Shows information about the version being accessed.

### 5. Warn if version is not enabled
```bash
if [ "$VERSION_STATE" != "ENABLED" ]; then
    echo "Warning: This version is $VERSION_STATE"
    read -p "Continue anyway? (y/N): " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi
```
Alerts user if accessing a disabled or destroyed version.

### 6. Access the secret value
```bash
echo -e "\n=== Secret Value ==="
gcloud secrets versions access $VERSION --secret=$SECRET_NAME
```
Retrieves and displays the actual secret value.

### 7. Provide usage examples
```bash
echo -e "\n\n=== Usage Examples ==="
echo "Export as environment variable:"
echo "  export MY_SECRET=\$(gcloud secrets versions access $VERSION --secret=$SECRET_NAME)"
echo ""
echo "Save to file:"
echo "  gcloud secrets versions access $VERSION --secret=$SECRET_NAME > secret.txt"
echo ""
echo "Use in script:"
echo "  API_KEY=\$(gcloud secrets versions access latest --secret=api-key)"
```
Shows common ways to use the secret value in practice.

### 8. Log access (optional)
```bash
echo -e "\n=== Access Logged ==="
echo "User: $(gcloud config get-value account)"
echo "Time: $(date)"
echo "Project: $(gcloud config get-value project)"
```
Records who accessed the secret and when for audit purposes.

## Validation
- Secret and version exist
- Version state is checked before access
- Secret value is retrieved successfully
- No errors during access operation

## Error Handling
- **"Secret not found"** - Check secret name and project
- **"Version not found"** - Use gcp-secrets-versions-list to see available versions
- **"Permission denied"** - Need secretmanager.versions.access permission
- **"Version is DESTROYED"** - Cannot access destroyed versions

## Safety Notes
- Be extremely careful with secret values - never commit them to git
- Avoid displaying secrets in logs or CI/CD output
- Clear terminal history after viewing secrets
- Consider piping to files or variables instead of displaying
- Use service accounts and IAM for production access

## Examples
- **Access latest version**
  ```
  gcp-secrets-versions-access database-password
  ```
  Retrieves the current version of the database password

- **Access specific version**
  ```
  gcp-secrets-versions-access api-key 3
  ```
  Retrieves version 3 of the api-key secret

- **Export to environment**
  ```
  export DB_PASS=$(gcp-secrets-versions-access db-password latest)
  ```
  Sets secret as environment variable

- **Save to file**
  ```
  gcp-secrets-versions-access ssl-cert latest > cert.pem
  ```
  Saves certificate to file