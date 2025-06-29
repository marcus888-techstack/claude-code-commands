# GCP Storage Download

## Purpose
Download files or directories from a Google Cloud Storage bucket to local storage

## Context
Use to retrieve data from Cloud Storage for local processing, backup restoration, or development. Supports downloading single files, multiple files with patterns, or entire directories.

## Parameters
- `$SOURCE` - Cloud Storage path to download from
  - Required
  - Example: `gs://my-bucket/file.txt`, `gs://my-bucket/folder/`
- `$DESTINATION` - Local directory or file path
  - Optional
  - Default: Current directory (`.`)
  - Example: `./downloads/`, `/tmp/file.txt`
- `$OPTIONS` - Additional download options
  - Optional
  - Example: `-r` (recursive), `-m` (parallel)

## Steps

### 1. Validate source exists
```bash
if ! gsutil ls $SOURCE &>/dev/null; then
    echo "Error: Source '$SOURCE' not found or not accessible"
    echo "Check path and permissions"
    exit 1
fi
```
Ensures the source path exists in Cloud Storage.

### 2. Set destination
```bash
if [ -z "$DESTINATION" ]; then
    DESTINATION="."
fi

# Create destination directory if needed
if [[ "$DESTINATION" == */ ]] || [ -d "$DESTINATION" ]; then
    mkdir -p "$DESTINATION"
fi
```
Prepares the local destination directory.

### 3. Analyze download content
```bash
echo "Download details:"
# Check if source is a file or directory
if gsutil ls -d $SOURCE | grep -q "/$"; then
    # Directory
    FILE_COUNT=$(gsutil ls -r $SOURCE | grep -v ":$" | wc -l)
    TOTAL_SIZE=$(gsutil du -sh $SOURCE | tail -1 | awk '{print $1}')
    echo "  Type: Directory"
    echo "  Files: $FILE_COUNT"
    echo "  Total size: $TOTAL_SIZE"
    DOWNLOAD_TYPE="directory"
else
    # Single file or pattern
    FILE_COUNT=$(gsutil ls $SOURCE 2>/dev/null | wc -l)
    if [ $FILE_COUNT -eq 1 ]; then
        SIZE=$(gsutil ls -l $SOURCE | tail -2 | head -1 | awk '{print $1}')
        SIZE_HR=$(numfmt --to=iec-i --suffix=B $SIZE 2>/dev/null || echo "${SIZE} bytes")
        echo "  Type: Single file"
        echo "  Size: $SIZE_HR"
        DOWNLOAD_TYPE="file"
    else
        echo "  Type: Multiple files"
        echo "  Files: $FILE_COUNT"
        DOWNLOAD_TYPE="pattern"
    fi
fi
echo "  Source: $SOURCE"
echo "  Destination: $DESTINATION"
```
Determines what will be downloaded.

### 4. Choose download method
```bash
# Use parallel download for multiple files
if [ "$DOWNLOAD_TYPE" = "directory" ] || [ $FILE_COUNT -gt 10 ]; then
    echo -e "\nUsing parallel download for better performance..."
    DOWNLOAD_CMD="gsutil -m cp -r"
else
    DOWNLOAD_CMD="gsutil cp"
fi
```
Selects optimal download method.

### 5. Check available disk space
```bash
if [ "$DOWNLOAD_TYPE" = "directory" ] || [ $FILE_COUNT -gt 1 ]; then
    REQUIRED_SPACE=$(gsutil du -s $SOURCE | awk '{print $1}')
    AVAILABLE_SPACE=$(df . | tail -1 | awk '{print $4}')
    
    if [ $REQUIRED_SPACE -gt $((AVAILABLE_SPACE * 1024)) ]; then
        echo "⚠️  Warning: May not have enough disk space"
        echo "Required: $(numfmt --to=iec-i --suffix=B $REQUIRED_SPACE)"
        echo "Available: $(numfmt --to=iec-i --suffix=B $((AVAILABLE_SPACE * 1024)))"
        read -p "Continue anyway? (y/N): " -n 1 -r
        echo
        if [[ ! $REPLY =~ ^[Yy]$ ]]; then
            exit 1
        fi
    fi
fi
```
Warns if insufficient disk space.

### 6. Perform the download
```bash
echo -e "\nDownloading..."
$DOWNLOAD_CMD $OPTIONS $SOURCE $DESTINATION

if [ $? -eq 0 ]; then
    echo "✓ Download completed successfully"
else
    echo "✗ Download failed"
    exit 1
fi
```
Executes the download with progress tracking.

### 7. Verify download
```bash
echo -e "\n=== Verifying Download ==="
if [ "$DOWNLOAD_TYPE" = "file" ]; then
    # Single file verification
    if [ -f "$DESTINATION" ]; then
        LOCAL_FILE="$DESTINATION"
    else
        FILENAME=$(basename $SOURCE)
        LOCAL_FILE="$DESTINATION/$FILENAME"
    fi
    
    if [ -f "$LOCAL_FILE" ]; then
        echo "✓ File downloaded: $LOCAL_FILE"
        ls -lh "$LOCAL_FILE"
    fi
elif [ "$DOWNLOAD_TYPE" = "directory" ]; then
    # Directory verification
    DOWNLOADED_COUNT=$(find "$DESTINATION" -type f | wc -l)
    echo "✓ Downloaded $DOWNLOADED_COUNT files to $DESTINATION"
    echo "Total size: $(du -sh "$DESTINATION" | cut -f1)"
fi
```
Confirms files were downloaded successfully.

### 8. Check file integrity (optional)
```bash
if [ "$DOWNLOAD_TYPE" = "file" ] && [ -f "$LOCAL_FILE" ]; then
    echo -e "\n=== Checking Integrity ==="
    # Get Cloud Storage MD5
    GCS_MD5=$(gsutil ls -L $SOURCE | grep "Hash (md5):" | awk '{print $3}')
    
    if [ -n "$GCS_MD5" ]; then
        # Calculate local MD5
        if command -v md5sum &> /dev/null; then
            LOCAL_MD5=$(md5sum "$LOCAL_FILE" | awk '{print $1}')
        elif command -v md5 &> /dev/null; then
            LOCAL_MD5=$(md5 -q "$LOCAL_FILE")
        fi
        
        if [ -n "$LOCAL_MD5" ]; then
            # Compare after base64 decoding GCS MD5
            GCS_MD5_HEX=$(echo $GCS_MD5 | base64 -d | xxd -p -c 256)
            if [ "$LOCAL_MD5" = "$GCS_MD5_HEX" ]; then
                echo "✓ File integrity verified"
            else
                echo "⚠️  MD5 mismatch - file may be corrupted"
            fi
        fi
    fi
fi
```
Optionally verifies file integrity.

### 9. Display download summary
```bash
echo -e "\n=== Download Summary ==="
if [ "$DOWNLOAD_TYPE" = "directory" ]; then
    echo "Downloaded files:"
    find "$DESTINATION" -type f -printf "  %p\n" 2>/dev/null || find "$DESTINATION" -type f | sed 's/^/  /'
else
    echo "Downloaded to: $DESTINATION"
fi

echo -e "\n=== Next Steps ==="
echo "List Cloud Storage contents:"
echo "  gsutil ls $SOURCE"
echo ""
if [ "$DOWNLOAD_TYPE" = "directory" ]; then
    echo "View downloaded files:"
    echo "  ls -la $DESTINATION"
fi
```
Shows what was downloaded and next steps.

## Validation
- Source exists in Cloud Storage
- Destination has sufficient space
- Download completes without errors
- Local files exist after download
- File sizes match (when possible)

## Error Handling
- **"Source not found"** - Check bucket/path exists and permissions
- **"Permission denied"** - Need storage.objects.get permission
- **"No space left"** - Free up disk space or change destination
- **"Network error"** - Retry with resume: `gsutil cp -c`
- **"Invalid path"** - Check for special characters in paths

## Safety Notes
- Large downloads may use significant bandwidth
- Check available disk space before downloading
- Use -n flag for dry run to preview
- Downloaded files inherit local filesystem permissions
- Consider virus scanning downloaded files

## Examples
- **Download single file**
  ```
  gcp-storage-download gs://my-bucket/report.pdf ./downloads/
  ```
  Downloads report.pdf to downloads directory

- **Download entire directory**
  ```
  gcp-storage-download gs://my-bucket/backups/ ./restore/ -r
  ```
  Downloads all files from backups folder

- **Download with pattern**
  ```
  gcp-storage-download "gs://my-bucket/logs/*.log" ./logs/ -m
  ```
  Downloads all .log files in parallel

- **Download to specific filename**
  ```
  gcp-storage-download gs://my-bucket/data.csv ./latest-data.csv
  ```
  Downloads and renames file locally