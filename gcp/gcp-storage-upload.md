# GCP Storage Upload

## Purpose
Upload files or directories to a Google Cloud Storage bucket

## Context
Use to upload data to Cloud Storage for backup, sharing, or serving content. Supports single files, multiple files, directories, and various upload options like compression and parallelization.

## Parameters
- `$SOURCE` - Local file or directory to upload
  - Required
  - Example: `./data.csv`, `/home/user/backups/`, `*.log`
- `$DESTINATION` - Target bucket and optional path
  - Required
  - Example: `gs://my-bucket/`, `gs://my-bucket/folder/`
- `$OPTIONS` - Additional upload options
  - Optional
  - Example: `-r` (recursive), `-m` (parallel), `-z` (gzip)

## Steps

### 1. Validate source exists
```bash
if [ ! -e "$SOURCE" ] && [ -z "$(ls $SOURCE 2>/dev/null)" ]; then
    echo "Error: Source '$SOURCE' not found"
    exit 1
fi
```
Ensures the source file or directory exists.

### 2. Validate destination bucket
```bash
BUCKET=$(echo $DESTINATION | sed 's|gs://||' | cut -d'/' -f1)
if ! gsutil ls -b gs://$BUCKET &>/dev/null; then
    echo "Error: Bucket 'gs://$BUCKET' not found or not accessible"
    echo "Create it with: gcp-storage-buckets-create $BUCKET"
    exit 1
fi
```
Verifies the destination bucket exists and is accessible.

### 3. Determine upload type
```bash
echo "Upload details:"
if [ -f "$SOURCE" ]; then
    SIZE=$(du -h "$SOURCE" | cut -f1)
    echo "  Type: Single file"
    echo "  Size: $SIZE"
elif [ -d "$SOURCE" ]; then
    FILE_COUNT=$(find "$SOURCE" -type f | wc -l)
    TOTAL_SIZE=$(du -sh "$SOURCE" | cut -f1)
    echo "  Type: Directory"
    echo "  Files: $FILE_COUNT"
    echo "  Total size: $TOTAL_SIZE"
else
    # Pattern match (e.g., *.txt)
    FILE_COUNT=$(ls $SOURCE 2>/dev/null | wc -l)
    echo "  Type: Pattern match"
    echo "  Files: $FILE_COUNT"
fi
echo "  Destination: $DESTINATION"
```
Analyzes what will be uploaded.

### 4. Choose upload method
```bash
# For large uploads or directories, use parallel upload
if [ -d "$SOURCE" ] || [ $(find $SOURCE -type f 2>/dev/null | wc -l) -gt 10 ]; then
    echo -e "\nUsing parallel upload for better performance..."
    UPLOAD_CMD="gsutil -m cp -r"
else
    UPLOAD_CMD="gsutil cp"
fi
```
Selects optimal upload method based on source.

### 5. Set upload options
```bash
# Add compression for text files
if [[ "$SOURCE" =~ \.(txt|log|csv|json|xml|html|js|css)$ ]]; then
    echo "Adding gzip compression for text files..."
    UPLOAD_OPTIONS="-z txt,log,csv,json,xml,html,js,css"
fi

# Preserve metadata
UPLOAD_OPTIONS="$UPLOAD_OPTIONS -p"
```
Configures upload options for optimization.

### 6. Perform the upload
```bash
echo -e "\nUploading..."
$UPLOAD_CMD $UPLOAD_OPTIONS $OPTIONS $SOURCE $DESTINATION

if [ $? -eq 0 ]; then
    echo "✓ Upload completed successfully"
else
    echo "✗ Upload failed"
    exit 1
fi
```
Executes the upload with progress tracking.

### 7. Verify upload
```bash
echo -e "\n=== Verifying Upload ==="
if [ -f "$SOURCE" ]; then
    # Single file verification
    FILENAME=$(basename "$SOURCE")
    if gsutil ls $DESTINATION$FILENAME &>/dev/null; then
        echo "✓ File uploaded: $DESTINATION$FILENAME"
        gsutil ls -l $DESTINATION$FILENAME
    fi
elif [ -d "$SOURCE" ]; then
    # Directory verification
    UPLOADED_COUNT=$(gsutil ls -r $DESTINATION | grep -v ":$" | wc -l)
    echo "✓ Uploaded $UPLOADED_COUNT files to $DESTINATION"
fi
```
Confirms files were uploaded successfully.

### 8. Set object metadata (optional)
```bash
read -p "Set public access for uploaded files? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    echo "Setting public access..."
    gsutil -m acl ch -r -u AllUsers:R $DESTINATION
    echo "✓ Files are now publicly accessible"
    
    # Show public URLs
    if [ -f "$SOURCE" ]; then
        FILENAME=$(basename "$SOURCE")
        echo "Public URL: https://storage.googleapis.com/$BUCKET/${DESTINATION#gs://$BUCKET/}$FILENAME"
    fi
fi
```
Optionally configures public access for uploaded files.

### 9. Display usage information
```bash
echo -e "\n=== Next Steps ==="
echo "List uploaded files:"
echo "  gsutil ls $DESTINATION"
echo ""
echo "Download files:"
echo "  gsutil cp $DESTINATION<filename> ."
echo ""
echo "Delete files:"
echo "  gsutil rm $DESTINATION<filename>"
```
Provides commands for managing uploaded files.

## Validation
- Source files exist
- Destination bucket is accessible
- Upload completes without errors
- Files appear in bucket listing
- File sizes match source

## Error Handling
- **"Source not found"** - Check file path and permissions
- **"Bucket not found"** - Verify bucket name and project
- **"Permission denied"** - Need storage.objects.create permission
- **"Quota exceeded"** - Check storage quotas
- **"Network error"** - Retry with resume option: `-c`

## Safety Notes
- Large uploads may incur bandwidth costs
- Use -n flag for dry run to preview actions
- Parallel uploads (-m) use more memory/CPU
- Consider encryption for sensitive data
- Check available storage space in bucket

## Examples
- **Upload single file**
  ```
  gcp-storage-upload report.pdf gs://my-bucket/reports/
  ```
  Uploads report.pdf to the reports folder

- **Upload directory recursively**
  ```
  gcp-storage-upload ./website/ gs://my-bucket/public/ -r
  ```
  Uploads entire website directory

- **Upload with pattern**
  ```
  gcp-storage-upload "*.log" gs://my-bucket/logs/ -m
  ```
  Uploads all .log files in parallel

- **Upload with compression**
  ```
  gcp-storage-upload large-file.json gs://my-bucket/ -z json
  ```
  Uploads with gzip compression