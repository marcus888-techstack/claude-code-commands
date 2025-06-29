# GCP IAM Service Accounts Create

## Purpose
Create a new service account for application authentication and authorization

## Context
Use to create service accounts that applications can use to authenticate with Google Cloud services. Service accounts provide a secure way for applications to access GCP resources without using user credentials.

## Parameters
- `$ACCOUNT_NAME` - Name for the service account
  - Required
  - Example: `app-backend`, `ci-deployer`
  - Must be 6-30 characters, lowercase letters, numbers, hyphens
- `$DISPLAY_NAME` - Human-readable display name
  - Optional
  - Default: Same as account name
  - Example: `"Application Backend Service"`
- `$DESCRIPTION` - Description of the service account's purpose
  - Optional
  - Example: `"Service account for backend API authentication"`

## Steps

### 1. Validate account name
```bash
# Validate service account name
if [[ ! "$ACCOUNT_NAME" =~ ^[a-z][a-z0-9-]{4,28}[a-z0-9]$ ]]; then
    echo "Error: Invalid service account name"
    echo "Rules:"
    echo "  - 6-30 characters"
    echo "  - Lowercase letters, numbers, and hyphens only"
    echo "  - Must start with a letter"
    echo "  - Must end with a letter or number"
    exit 1
fi

PROJECT=$(gcloud config get-value project)
FULL_EMAIL="${ACCOUNT_NAME}@${PROJECT}.iam.gserviceaccount.com"
```
Ensures valid service account naming.

### 2. Check if account already exists
```bash
if gcloud iam service-accounts describe $FULL_EMAIL &>/dev/null; then
    echo "Error: Service account '$FULL_EMAIL' already exists"
    exit 1
fi

echo "Creating service account:"
echo "  Name: $ACCOUNT_NAME"
echo "  Email: $FULL_EMAIL"
echo "  Project: $PROJECT"
```
Prevents duplicate service accounts.

### 3. Set display name and description
```bash
DISPLAY_NAME=${DISPLAY_NAME:-$ACCOUNT_NAME}
DESCRIPTION=${DESCRIPTION:-"Service account created on $(date +%Y-%m-%d)"}

echo "  Display name: $DISPLAY_NAME"
echo "  Description: $DESCRIPTION"
```
Configures metadata for the service account.

### 4. Create the service account
```bash
echo -e "\n=== Creating Service Account ==="
gcloud iam service-accounts create $ACCOUNT_NAME \
    --display-name="$DISPLAY_NAME" \
    --description="$DESCRIPTION"

if [ $? -eq 0 ]; then
    echo "✓ Service account created successfully"
else
    echo "✗ Failed to create service account"
    exit 1
fi
```
Creates the service account.

### 5. Verify creation
```bash
echo -e "\n=== Verification ==="
gcloud iam service-accounts describe $FULL_EMAIL --format="yaml(email,displayName,projectId,uniqueId)"
```
Confirms service account details.

### 6. Create and download key (optional)
```bash
echo -e "\n=== Service Account Key ==="
read -p "Create and download JSON key file? (y/N): " -n 1 -r
echo

if [[ $REPLY =~ ^[Yy]$ ]]; then
    KEY_FILE="${ACCOUNT_NAME}-key.json"
    
    echo "Creating key file: $KEY_FILE"
    gcloud iam service-accounts keys create $KEY_FILE \
        --iam-account=$FULL_EMAIL
    
    if [ -f "$KEY_FILE" ]; then
        echo "✓ Key file created: $KEY_FILE"
        echo "⚠️  IMPORTANT: Store this key file securely!"
        echo "  - Do not commit to version control"
        echo "  - Set appropriate file permissions"
        echo "  - Consider using Secret Manager instead"
        
        # Set restrictive permissions
        chmod 600 $KEY_FILE
        echo "✓ File permissions set to 600 (read/write for owner only)"
    fi
else
    echo "Skipping key creation"
    echo "To create a key later:"
    echo "  gcloud iam service-accounts keys create KEY_FILE --iam-account=$FULL_EMAIL"
fi
```
Optionally creates authentication key.

### 7. Suggest common role assignments
```bash
echo -e "\n=== Suggested Role Assignments ==="
echo "Common roles for service accounts:"
echo ""
echo "# For compute instances:"
echo "gcloud projects add-iam-policy-binding $PROJECT \\"
echo "    --member=serviceAccount:$FULL_EMAIL \\"
echo "    --role=roles/compute.instanceAdmin"
echo ""
echo "# For storage access:"
echo "gcloud projects add-iam-policy-binding $PROJECT \\"
echo "    --member=serviceAccount:$FULL_EMAIL \\"
echo "    --role=roles/storage.objectAdmin"
echo ""
echo "# For Cloud Run deployment:"
echo "gcloud projects add-iam-policy-binding $PROJECT \\"
echo "    --member=serviceAccount:$FULL_EMAIL \\"
echo "    --role=roles/run.developer"
echo ""
echo "# For Secret Manager access:"
echo "gcloud projects add-iam-policy-binding $PROJECT \\"
echo "    --member=serviceAccount:$FULL_EMAIL \\"
echo "    --role=roles/secretmanager.secretAccessor"
```
Provides role assignment examples.

### 8. Show usage instructions
```bash
echo -e "\n=== Usage Instructions ==="
cat << EOF
1. Authenticate using key file:
   export GOOGLE_APPLICATION_CREDENTIALS="$KEY_FILE"

2. Use with gcloud:
   gcloud auth activate-service-account --key-file=$KEY_FILE

3. Attach to Compute Engine instance:
   gcloud compute instances create INSTANCE_NAME \\
       --service-account=$FULL_EMAIL \\
       --scopes=cloud-platform

4. Use with Cloud Run:
   gcloud run deploy SERVICE_NAME \\
       --service-account=$FULL_EMAIL

5. Grant impersonation permission:
   gcloud iam service-accounts add-iam-policy-binding $FULL_EMAIL \\
       --member=user:YOUR_EMAIL \\
       --role=roles/iam.serviceAccountUser
EOF
```
Provides practical usage examples.

### 9. Set up monitoring
```bash
echo -e "\n=== Monitoring Setup ==="
echo "To monitor service account usage:"
echo "1. View recent activity:"
echo "   gcloud logging read 'protoPayload.authenticationInfo.principalEmail=\"$FULL_EMAIL\"' --limit=10"
echo ""
echo "2. Set up alerts for key usage:"
echo "   - Go to Cloud Console > Monitoring > Alerting"
echo "   - Create alert for service account key usage"
echo ""
echo "3. Audit service account permissions regularly:"
echo "   gcloud projects get-iam-policy $PROJECT --flatten='bindings[].members' --filter='bindings.members:$FULL_EMAIL'"
```
Suggests monitoring configuration.

## Validation
- Service account is created with correct email format
- Display name and description are set
- Can describe the service account
- Key file is created with proper permissions (if requested)

## Error Handling
- **"Invalid account name"** - Follow naming conventions
- **"Already exists"** - Choose a different name
- **"Permission denied"** - Need iam.serviceAccounts.create permission
- **"Quota exceeded"** - Check service account quota (default: 100)

## Safety Notes
- Service account keys are sensitive - store securely
- Never commit keys to version control
- Prefer workload identity over keys when possible
- Regularly rotate keys
- Monitor service account usage
- Apply principle of least privilege for roles

## Examples
- **Create basic service account**
  ```
  gcp-iam-service-accounts-create app-backend
  ```
  Creates service account with default settings

- **Create with display name**
  ```
  gcp-iam-service-accounts-create ci-deployer "CI/CD Deployment Account"
  ```
  Creates with custom display name

- **Create with full details**
  ```
  gcp-iam-service-accounts-create data-processor "Data Processing Service" "Processes daily data uploads from partners"
  ```
  Creates with name, display name, and description