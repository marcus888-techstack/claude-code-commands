# GCP Storage Buckets Create

## Purpose
Create a new Cloud Storage bucket with specified configuration options

## Context
Use to create storage buckets for storing objects like files, backups, or static website content. Buckets are globally unique containers for your data with configurable location, storage class, and access controls.

## Parameters
- `$BUCKET_NAME` - Globally unique name for the bucket
  - Required
  - Example: `my-app-backups-12345`
- `$LOCATION` - Geographic location for the bucket
  - Optional
  - Default: `US`
  - Example: `us-central1`, `EU`, `ASIA`
- `$STORAGE_CLASS` - Storage class for cost optimization
  - Optional
  - Default: `STANDARD`
  - Example: `NEARLINE`, `COLDLINE`, `ARCHIVE`

## Steps

### 1. Validate bucket name
```bash
# Bucket naming rules
if ! [[ "$BUCKET_NAME" =~ ^[a-z0-9][a-z0-9._-]*[a-z0-9]$ ]]; then
    echo "Error: Invalid bucket name '$BUCKET_NAME'"
    echo "Rules: lowercase letters, numbers, hyphens, underscores, dots"
    echo "Must start and end with letter or number"
    exit 1
fi

if [ ${#BUCKET_NAME} -lt 3 ] || [ ${#BUCKET_NAME} -gt 63 ]; then
    echo "Error: Bucket name must be 3-63 characters"
    exit 1
fi
```
Ensures bucket name follows Google Cloud naming conventions.

### 2. Set defaults
```bash
LOCATION=${LOCATION:-US}
STORAGE_CLASS=${STORAGE_CLASS:-STANDARD}
PROJECT=$(gcloud config get-value project)

echo "Creating bucket:"
echo "  Name: gs://$BUCKET_NAME"
echo "  Location: $LOCATION"
echo "  Storage class: $STORAGE_CLASS"
echo "  Project: $PROJECT"
```
Sets default values and displays configuration.

### 3. Check if bucket name is available
```bash
echo -e "\nChecking bucket name availability..."
if gsutil ls -b gs://$BUCKET_NAME &>/dev/null; then
    echo "Error: Bucket name 'gs://$BUCKET_NAME' is already taken"
    echo "Bucket names are globally unique across all Google Cloud projects"
    exit 1
fi
echo "✓ Bucket name is available"
```
Verifies the bucket name isn't already in use globally.

### 4. Create the bucket
```bash
echo -e "\nCreating bucket..."
gsutil mb -p $PROJECT -c $STORAGE_CLASS -l $LOCATION gs://$BUCKET_NAME
```
Creates the bucket with specified configuration.

### 5. Set bucket labels
```bash
echo -e "\nAdding metadata labels..."
gsutil label ch -l created-by:$(whoami) gs://$BUCKET_NAME
gsutil label ch -l created-date:$(date +%Y-%m-%d) gs://$BUCKET_NAME
gsutil label ch -l environment:production gs://$BUCKET_NAME
```
Adds labels for organization and tracking.

### 6. Configure bucket settings
```bash
echo -e "\n=== Configuring Bucket Settings ==="

# Enable versioning for data protection
read -p "Enable versioning? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    gsutil versioning set on gs://$BUCKET_NAME
    echo "✓ Versioning enabled"
fi

# Set lifecycle rules
read -p "Add lifecycle rule to delete objects after 365 days? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    cat > /tmp/lifecycle.json <<EOF
{
  "lifecycle": {
    "rule": [{
      "action": {"type": "Delete"},
      "condition": {"age": 365}
    }]
  }
}
EOF
    gsutil lifecycle set /tmp/lifecycle.json gs://$BUCKET_NAME
    rm /tmp/lifecycle.json
    echo "✓ Lifecycle policy configured"
fi
```
Optionally configures versioning and lifecycle policies.

### 7. Set access controls
```bash
echo -e "\n=== Access Control ==="
# Default is uniform bucket-level access (recommended)
gsutil iam ch projectOwner:$PROJECT:objectAdmin gs://$BUCKET_NAME
echo "✓ Bucket-level access configured"
echo "Note: Using uniform bucket-level access (recommended)"
```
Configures access control settings.

### 8. Display bucket information
```bash
echo -e "\n=== Bucket Created Successfully ==="
gsutil ls -L -b gs://$BUCKET_NAME
```
Shows detailed information about the newly created bucket.

### 9. Provide usage examples
```bash
echo -e "\n=== Usage Examples ==="
echo "Upload file:"
echo "  gsutil cp file.txt gs://$BUCKET_NAME/"
echo ""
echo "List contents:"
echo "  gsutil ls gs://$BUCKET_NAME/"
echo ""
echo "Make object public:"
echo "  gsutil acl ch -u AllUsers:R gs://$BUCKET_NAME/file.txt"
echo ""
echo "Delete bucket (when empty):"
echo "  gsutil rb gs://$BUCKET_NAME"
```
Provides common commands for using the bucket.

## Validation
- Bucket is created successfully
- Can list the bucket
- Labels are applied
- Settings are configured as requested

## Error Handling
- **"Bucket name already exists"** - Choose a different, globally unique name
- **"Invalid bucket name"** - Follow naming rules (lowercase, no spaces)
- **"Permission denied"** - Need storage.buckets.create permission
- **"Invalid location"** - Use valid location like US, EU, us-central1
- **"Billing not enabled"** - Enable billing for the project

## Safety Notes
- Bucket names are globally unique and public
- Don't include sensitive information in bucket names
- Consider encryption settings for sensitive data
- Review access controls before storing data
- Storage costs apply immediately

## Examples
- **Create standard bucket**
  ```
  gcp-storage-buckets-create my-app-data-12345
  ```
  Creates bucket in US with STANDARD storage class

- **Create regional bucket**
  ```
  gcp-storage-buckets-create my-backups-12345 us-central1 NEARLINE
  ```
  Creates NEARLINE bucket in specific region

- **Create archive bucket**
  ```
  gcp-storage-buckets-create my-archives-12345 EU ARCHIVE
  ```
  Creates ARCHIVE bucket in Europe for long-term storage