# GCP Run Services List

## Purpose
List all Cloud Run services across regions with their status and configuration

## Context
Use to view all deployed Cloud Run services, their URLs, traffic allocation, and resource usage. Essential for managing serverless applications and understanding what's deployed across different regions.

## Parameters
- `$ARGUMENTS` - Optional filters or format options
  - Optional
  - Example: `--region=us-central1`, `--limit=50`, `--format=json`

## Steps

### 1. Check Cloud Run API status
```bash
if ! gcloud services list --enabled --filter="name:run.googleapis.com" --format="value(name)" | grep -q run; then
    echo "Warning: Cloud Run API is not enabled"
    echo "Enable it with: gcloud services enable run.googleapis.com"
    exit 1
fi
```
Ensures Cloud Run API is enabled.

### 2. Display project context
```bash
echo "Project: $(gcloud config get-value project)"
echo "========================================"
```
Shows which project's services are being listed.

### 3. List services across all regions
```bash
# If no region specified, list from all regions
if [[ ! "$ARGUMENTS" =~ --region ]]; then
    echo "Listing Cloud Run services across all regions..."
    
    # Get list of available regions
    REGIONS=$(gcloud run regions list --format="value(name)")
    
    for region in $REGIONS; do
        SERVICE_COUNT=$(gcloud run services list --region=$region --format="value(name)" 2>/dev/null | wc -l)
        
        if [ $SERVICE_COUNT -gt 0 ]; then
            echo -e "\n=== Region: $region ==="
            gcloud run services list --region=$region --format="table(
                name,
                status.url,
                spec.template.spec.containers[0].image:label=IMAGE,
                status.latestReadyRevisionName:label=REVISION
            )" $ARGUMENTS
        fi
    done
else
    # List from specified region
    gcloud run services list --format="table(
        name,
        region,
        status.url,
        spec.template.spec.containers[0].image:label=IMAGE,
        status.latestReadyRevisionName:label=REVISION
    )" $ARGUMENTS
fi
```
Lists all services with key information.

### 4. Show service statistics
```bash
echo -e "\n========================================"
TOTAL_SERVICES=0
TOTAL_REGIONS=0

for region in $(gcloud run regions list --format="value(name)"); do
    COUNT=$(gcloud run services list --region=$region --format="value(name)" 2>/dev/null | wc -l)
    if [ $COUNT -gt 0 ]; then
        TOTAL_SERVICES=$((TOTAL_SERVICES + COUNT))
        TOTAL_REGIONS=$((TOTAL_REGIONS + 1))
    fi
done

echo "Total services: $TOTAL_SERVICES"
echo "Regions with services: $TOTAL_REGIONS"
```
Provides summary statistics.

### 5. Display services by authentication
```bash
echo -e "\n=== Services by Authentication ==="
PUBLIC_COUNT=0
PRIVATE_COUNT=0

for region in $(gcloud run regions list --format="value(name)"); do
    for service in $(gcloud run services list --region=$region --format="value(name)" 2>/dev/null); do
        IAM_POLICY=$(gcloud run services get-iam-policy $service --region=$region 2>/dev/null)
        
        if echo "$IAM_POLICY" | grep -q "allUsers"; then
            PUBLIC_COUNT=$((PUBLIC_COUNT + 1))
        else
            PRIVATE_COUNT=$((PRIVATE_COUNT + 1))
        fi
    done
done

echo "Public services: $PUBLIC_COUNT"
echo "Private services: $PRIVATE_COUNT"
```
Shows authentication status of services.

### 6. List services with traffic information
```bash
echo -e "\n=== Traffic Allocation (Sample) ==="
# Show traffic info for up to 5 services
SAMPLE_COUNT=0

for region in $(gcloud run regions list --format="value(name)"); do
    for service in $(gcloud run services list --region=$region --format="value(name)" 2>/dev/null); do
        if [ $SAMPLE_COUNT -lt 5 ]; then
            echo -e "\n$service ($region):"
            gcloud run services describe $service --region=$region --format="table(
                spec.traffic[].revisionName,
                spec.traffic[].percent
            )" 2>/dev/null | tail -n +2
            SAMPLE_COUNT=$((SAMPLE_COUNT + 1))
        fi
    done
done
```
Shows traffic distribution for sample services.

### 7. Display resource usage
```bash
echo -e "\n=== Resource Configuration ==="
echo "Memory and CPU limits for services:"

for region in $(gcloud run regions list --format="value(name)"); do
    gcloud run services list --region=$region --format="table(
        name,
        spec.template.spec.containers[0].resources.limits.memory:label=MEMORY,
        spec.template.spec.containers[0].resources.limits.cpu:label=CPU
    )" 2>/dev/null | grep -v "^$" | tail -n +2
done | sort -u
```
Shows resource allocation for services.

### 8. Check for outdated services
```bash
echo -e "\n=== Service Age Analysis ==="
THIRTY_DAYS_AGO=$(date -d "30 days ago" +%Y-%m-%d 2>/dev/null || date -v-30d +%Y-%m-%d)

echo "Services not updated in 30+ days:"
for region in $(gcloud run regions list --format="value(name)"); do
    gcloud run services list --region=$region \
        --format="table(name,metadata.labels.updateTime)" \
        --filter="metadata.labels.updateTime<$THIRTY_DAYS_AGO" 2>/dev/null
done
```
Identifies potentially outdated services.

### 9. Display cost optimization tips
```bash
echo -e "\n=== Cost Optimization ==="
echo "Tips for reducing Cloud Run costs:"
echo "- Services scale to zero when not in use (no charges)"
echo "- Set appropriate memory limits (default is often too high)"
echo "- Use minimum instances = 0 for dev/test services"
echo "- Consider using Cloud Run jobs for batch workloads"
echo ""
echo "To check current pricing:"
echo "  https://cloud.google.com/run/pricing"
```
Provides cost optimization guidance.

## Validation
- Cloud Run API is enabled
- Service list displays correctly
- URLs are properly formatted
- Region information is accurate

## Error Handling
- **"API not enabled"** - Enable with `gcloud services enable run.googleapis.com`
- **"Permission denied"** - Need run.services.list permission
- **"No services found"** - No Cloud Run services deployed
- **"Invalid region"** - Check region name format

## Safety Notes
- Read-only operation, safe to run frequently
- Listing across all regions may take time
- No impact on running services
- URLs shown may be public endpoints

## Examples
- **List all services**
  ```
  gcp-run-services-list
  ```
  Shows all services across all regions

- **List in specific region**
  ```
  gcp-run-services-list "--region=us-central1"
  ```
  Shows services in us-central1 only

- **List with custom format**
  ```
  gcp-run-services-list "--format=json"
  ```
  Returns service data in JSON format

- **List recent services**
  ```
  gcp-run-services-list "--limit=10 --sort-by=~metadata.creationTimestamp"
  ```
  Shows 10 most recently created services