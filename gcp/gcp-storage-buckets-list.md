# GCP Storage Buckets List

## Purpose
List all Cloud Storage buckets in the current project with their configuration and usage details

## Context
Use to view all storage buckets, their locations, storage classes, and usage statistics. Essential for managing cloud storage resources and understanding storage costs.

## Parameters
- `$ARGUMENTS` - Optional filters or format options
  - Optional
  - Example: `--filter=location:US`, `--format=json`

## Steps

### 1. Check Storage API status
```bash
if ! gcloud services list --enabled --filter="name:storage.googleapis.com" --format="value(name)" | grep -q storage; then
    echo "Warning: Cloud Storage API is not enabled"
    echo "Enable it with: gcloud services enable storage.googleapis.com"
    exit 1
fi
```
Ensures the Cloud Storage API is enabled.

### 2. Display project context
```bash
echo "Project: $(gcloud config get-value project)"
echo "========================================"
```
Shows which project's buckets are being listed.

### 3. List all buckets
```bash
gsutil ls -L -b $ARGUMENTS | grep -E "gs://|Location:|Storage class:|Lifecycle management:|Versioning:|Total size:|Objects count:" | \
awk '
    /^gs:/ {
        if (bucket) print "";
        bucket = $1;
        print "Bucket: " bucket
    }
    /Location:/ {print "  " $0}
    /Storage class:/ {print "  " $0}
    /Versioning:/ {print "  " $0}
    /Lifecycle management:/ {print "  " $0}
'
```
Lists buckets with key configuration details.

### 4. Show bucket count and summary
```bash
echo -e "\n========================================"
BUCKET_COUNT=$(gsutil ls | wc -l)
echo "Total buckets: $BUCKET_COUNT"
```
Provides total bucket count.

### 5. Display storage usage summary
```bash
echo -e "\n=== Storage Usage Summary ==="
TOTAL_SIZE=0
TOTAL_OBJECTS=0

for bucket in $(gsutil ls); do
    # Get bucket size and object count
    BUCKET_INFO=$(gsutil du -s $bucket 2>/dev/null | tail -1)
    if [ -n "$BUCKET_INFO" ]; then
        SIZE=$(echo $BUCKET_INFO | awk '{print $1}')
        TOTAL_SIZE=$((TOTAL_SIZE + SIZE))
        
        OBJECT_COUNT=$(gsutil ls -r $bucket | grep -v "/:$" | wc -l)
        TOTAL_OBJECTS=$((TOTAL_OBJECTS + OBJECT_COUNT))
        
        # Convert size to human readable
        SIZE_HR=$(numfmt --to=iec-i --suffix=B $SIZE 2>/dev/null || echo "${SIZE} bytes")
        echo "$bucket: $SIZE_HR ($OBJECT_COUNT objects)"
    fi
done

echo -e "\nTotal storage used: $(numfmt --to=iec-i --suffix=B $TOTAL_SIZE 2>/dev/null || echo "${TOTAL_SIZE} bytes")"
echo "Total objects: $TOTAL_OBJECTS"
```
Shows storage usage per bucket and totals.

### 6. Group buckets by location
```bash
echo -e "\n=== Buckets by Location ==="
for location in $(gsutil ls -L -b | grep "Location:" | awk '{print $NF}' | sort -u); do
    COUNT=$(gsutil ls -L -b | grep -B1 "Location:.*$location" | grep "gs://" | wc -l)
    echo "$location: $COUNT bucket(s)"
done
```
Groups buckets by their geographic location.

### 7. Show lifecycle and retention policies
```bash
echo -e "\n=== Bucket Policies ==="
for bucket in $(gsutil ls | head -5); do
    echo -n "$bucket: "
    
    # Check lifecycle
    if gsutil lifecycle get $bucket 2>/dev/null | grep -q "lifecycle"; then
        echo -n "Lifecycle ✓ "
    fi
    
    # Check retention
    if gsutil retention get $bucket 2>/dev/null | grep -q "Retention Policy"; then
        echo -n "Retention ✓ "
    fi
    
    # Check versioning
    if gsutil versioning get $bucket 2>/dev/null | grep -q "Enabled"; then
        echo -n "Versioning ✓"
    fi
    
    echo ""
done
```
Shows which buckets have policies configured.

### 8. List public buckets
```bash
echo -e "\n=== Public Access Check ==="
PUBLIC_COUNT=0
for bucket in $(gsutil ls); do
    if gsutil iam get $bucket 2>/dev/null | grep -q "allUsers\|allAuthenticatedUsers"; then
        echo "⚠️  $bucket has public access"
        PUBLIC_COUNT=$((PUBLIC_COUNT + 1))
    fi
done

if [ $PUBLIC_COUNT -eq 0 ]; then
    echo "✓ No buckets with public access found"
else
    echo "⚠️  Warning: $PUBLIC_COUNT bucket(s) with public access"
fi
```
Identifies buckets with public access for security review.

## Validation
- Storage API is enabled
- Bucket list is displayed
- Bucket names follow gs:// format
- Location and storage class are shown

## Error Handling
- **"API not enabled"** - Enable with `gcloud services enable storage.googleapis.com`
- **"Permission denied"** - Need storage.buckets.list permission
- **"No buckets found"** - No buckets exist in the project
- **"AccessDeniedException"** - Check IAM permissions

## Safety Notes
- Read-only operation, safe to run frequently
- Large buckets may take time to calculate size
- Public access check helps identify security risks
- Storage costs apply to bucket contents

## Examples
- **List all buckets**
  ```
  gcp-storage-buckets-list
  ```
  Shows all buckets with details

- **List buckets in US**
  ```
  gcp-storage-buckets-list "--filter=location:US"
  ```
  Shows only buckets in US locations

- **List as JSON**
  ```
  gcp-storage-buckets-list "--format=json"
  ```
  Returns bucket list in JSON format

- **Quick bucket names only**
  ```
  gsutil ls
  ```
  Simple list of bucket names