# GCP Regions List

## Purpose
List all available Google Cloud regions and zones with their locations and status

## Context
Use to discover available regions and zones for deploying resources. Shows geographic locations, available services, and helps choose the best region for your workloads based on location, features, and compliance requirements.

## Parameters
- `$ARGUMENTS` - Optional filters or format options
  - Optional
  - Example: `--filter=name:us-*`, `--format=json`
- `$SHOW_ZONES` - Whether to show zones within regions
  - Optional
  - Default: `yes`
  - Options: `yes`, `no`

## Steps

### 1. Display project context
```bash
echo "=== Google Cloud Regions and Zones ==="
echo "Project: $(gcloud config get-value project)"
CURRENT_REGION=$(gcloud config get-value compute/region 2>/dev/null || echo "Not set")
CURRENT_ZONE=$(gcloud config get-value compute/zone 2>/dev/null || echo "Not set")
echo "Current region: $CURRENT_REGION"
echo "Current zone: $CURRENT_ZONE"
echo "========================================"
```
Shows current configuration context.

### 2. List all regions
```bash
echo -e "\n=== Available Regions ==="
gcloud compute regions list --format="table(
    name,
    location,
    status
)" $ARGUMENTS
```
Displays all regions with locations.

### 3. Show region statistics
```bash
echo -e "\n=== Region Summary ==="
TOTAL_REGIONS=$(gcloud compute regions list --format="value(name)" | wc -l)
echo "Total regions: $TOTAL_REGIONS"

echo -e "\nRegions by continent:"
gcloud compute regions list --format="value(name)" | \
    sed -E 's/^(asia|europe|us|northamerica|southamerica|australia).*/\1/' | \
    sort | uniq -c | sort -rn | \
    while read count continent; do
        printf "  %-15s %d regions\n" "$continent" "$count"
    done
```
Provides regional distribution summary.

### 4. List zones if requested
```bash
if [ "${SHOW_ZONES:-yes}" = "yes" ]; then
    echo -e "\n=== Available Zones ==="
    gcloud compute zones list --format="table(
        name,
        region.scope(),
        status
    )" $ARGUMENTS
    
    TOTAL_ZONES=$(gcloud compute zones list --format="value(name)" | wc -l)
    echo -e "\nTotal zones: $TOTAL_ZONES"
fi
```
Shows all zones grouped by region.

### 5. Display detailed region information
```bash
echo -e "\n=== Region Details (Sample) ==="
# Show details for a few regions
for region in $(gcloud compute regions list --format="value(name)" --limit=3); do
    echo -e "\nRegion: $region"
    REGION_INFO=$(gcloud compute regions describe $region --format="yaml(
        name,
        selfLink,
        zones,
        quotas[0:3]
    )" 2>/dev/null)
    
    # Extract zone count
    ZONE_COUNT=$(echo "$REGION_INFO" | grep -c "zones/.*/zones/")
    echo "  Zones: $ZONE_COUNT"
    
    # Show sample quotas
    echo "  Sample quotas:"
    echo "$REGION_INFO" | grep -A1 "metric:" | grep -E "(metric|limit):" | \
        paste - - | sed 's/.*metric: //;s/- limit:/:/;s/^/    /'
done
```
Shows detailed information for sample regions.

### 6. Show regions with specific features
```bash
echo -e "\n=== Regions by Features ==="
echo "Multi-region locations:"
echo "  US (United States)"
echo "  EU (European Union)"
echo "  ASIA (Asia Pacific)"
echo ""
echo "Regions with 4 zones:"
gcloud compute regions list --format="value(name)" | \
while read region; do
    ZONE_COUNT=$(gcloud compute zones list --filter="region:$region" --format="value(name)" | wc -l)
    if [ $ZONE_COUNT -ge 4 ]; then
        echo "  $region ($ZONE_COUNT zones)"
    fi
done
```
Highlights regions with specific characteristics.

### 7. Show pricing tiers
```bash
echo -e "\n=== Pricing Information ==="
cat << 'EOF'
Region pricing tiers (relative to US regions):
  
Tier 1 (Lowest cost):
  - US regions (us-central1, us-east1, us-west1)
  
Tier 2 (Moderate cost):
  - Europe regions
  - Asia regions (asia-east1, asia-northeast1)
  
Tier 3 (Higher cost):
  - Australia regions
  - South America regions
  - Some Asia regions

Note: Actual pricing varies by service. Check:
https://cloud.google.com/compute/pricing
EOF
```
Provides pricing tier information.

### 8. Suggest region selection
```bash
echo -e "\n=== Region Selection Guide ==="
cat << 'EOF'
Consider these factors when choosing a region:

1. Latency - Choose closest to your users
2. Compliance - Data residency requirements
3. Service availability - Not all services in all regions
4. Pricing - Varies by region
5. Redundancy - Multi-zone regions for HA
6. Network - Proximity to other GCP services

Common patterns:
- Web apps: Multiple regions for global users
- Data processing: Same region as data storage
- Development: Closest region to team
- Compliance: Specific regions for regulations
EOF
```
Provides region selection guidance.

### 9. Show how to set default region/zone
```bash
echo -e "\n=== Set Default Region/Zone ==="
echo "To set defaults for your project:"
echo ""
echo "# Set default region"
echo "gcloud config set compute/region REGION_NAME"
echo "Example: gcloud config set compute/region us-central1"
echo ""
echo "# Set default zone"
echo "gcloud config set compute/zone ZONE_NAME"
echo "Example: gcloud config set compute/zone us-central1-a"
echo ""
echo "# View current defaults"
echo "gcloud config list compute/"
```
Shows how to configure defaults.

## Validation
- Regions are listed with valid names
- Status shows as UP for available regions
- Zone counts are accurate
- Location descriptions are present

## Error Handling
- **"API not enabled"** - Enable Compute Engine API
- **"Permission denied"** - Need compute.regions.list permission
- **"Invalid filter"** - Check filter syntax

## Safety Notes
- Read-only operation
- No impact on resources
- Some regions may have limited availability
- New regions are added periodically

## Examples
- **List all regions and zones**
  ```
  gcp-regions-list
  ```
  Shows complete list of regions and zones

- **List regions only**
  ```
  gcp-regions-list "" no
  ```
  Shows regions without zone details

- **List US regions only**
  ```
  gcp-regions-list "--filter=name:us-*"
  ```
  Shows only US regions

- **List as JSON**
  ```
  gcp-regions-list "--format=json" no
  ```
  Returns region data in JSON format