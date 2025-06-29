# GCP GKE Clusters List

## Purpose
List all Google Kubernetes Engine (GKE) clusters across regions with their configuration and status

## Context
Use to view all Kubernetes clusters in your project, their locations, versions, node counts, and status. Essential for managing container orchestration infrastructure and understanding cluster resources.

## Parameters
- `$ARGUMENTS` - Optional filters or format options
  - Optional
  - Example: `--filter=status:RUNNING`, `--region=us-central1`

## Steps

### 1. Check Container API status
```bash
if ! gcloud services list --enabled --filter="name:container.googleapis.com" --format="value(name)" | grep -q container; then
    echo "Warning: Kubernetes Engine API is not enabled"
    echo "Enable it with: gcloud services enable container.googleapis.com"
    exit 1
fi
```
Ensures the Kubernetes Engine API is enabled.

### 2. Display project context
```bash
echo "Project: $(gcloud config get-value project)"
echo "Listing all GKE clusters..."
echo "========================================"
```
Shows which project's clusters are being listed.

### 3. List all clusters
```bash
gcloud container clusters list --format="table(
    name,
    location,
    master_version,
    nodeConfig.machineType,
    currentNodeCount,
    status
)" $ARGUMENTS
```
Displays all clusters with key information.

### 4. Show cluster statistics
```bash
echo "========================================"
TOTAL=$(gcloud container clusters list --format="value(name)" | wc -l)
RUNNING=$(gcloud container clusters list --filter="status:RUNNING" --format="value(name)" | wc -l)
DEGRADED=$(gcloud container clusters list --filter="status:DEGRADED" --format="value(name)" | wc -l)

echo "Total clusters: $TOTAL"
echo "Running: $RUNNING"
if [ $DEGRADED -gt 0 ]; then
    echo "⚠️  Degraded: $DEGRADED"
fi
```
Provides summary statistics about cluster states.

### 5. Group clusters by location
```bash
echo -e "\n=== Clusters by Location ==="
for location in $(gcloud container clusters list --format="value(location)" | sort -u); do
    COUNT=$(gcloud container clusters list --filter="location:$location" --format="value(name)" | wc -l)
    TYPE=$(echo $location | grep -q "-[a-z]$" && echo "Zone" || echo "Region")
    echo "$location ($TYPE): $COUNT cluster(s)"
done
```
Shows distribution across zones and regions.

### 6. Display version information
```bash
echo -e "\n=== Kubernetes Versions ==="
gcloud container clusters list --format="value(masterVersion)" | sort | uniq -c | sort -rn | \
while read count version; do
    echo "  $version: $count cluster(s)"
done
```
Shows Kubernetes version distribution.

### 7. Calculate total resources
```bash
echo -e "\n=== Resource Summary ==="
TOTAL_NODES=0
TOTAL_CPUS=0
TOTAL_MEMORY=0

for cluster in $(gcloud container clusters list --format="value(name,location)" | tr '\t' ':'); do
    CLUSTER_NAME=$(echo $cluster | cut -d: -f1)
    CLUSTER_LOCATION=$(echo $cluster | cut -d: -f2)
    
    # Get cluster details
    CLUSTER_INFO=$(gcloud container clusters describe $CLUSTER_NAME --location=$CLUSTER_LOCATION --format="value(currentNodeCount,nodeConfig.machineType)" 2>/dev/null)
    
    if [ -n "$CLUSTER_INFO" ]; then
        NODE_COUNT=$(echo $CLUSTER_INFO | awk '{print $1}')
        TOTAL_NODES=$((TOTAL_NODES + NODE_COUNT))
    fi
done

echo "Total nodes across all clusters: $TOTAL_NODES"
```
Aggregates resource usage across clusters.

### 8. Check cluster health
```bash
echo -e "\n=== Cluster Health Check ==="
for cluster in $(gcloud container clusters list --format="value(name,location,status)" | head -5); do
    NAME=$(echo $cluster | awk '{print $1}')
    LOCATION=$(echo $cluster | awk '{print $2}')
    STATUS=$(echo $cluster | awk '{print $3}')
    
    echo -n "$NAME: $STATUS"
    
    if [ "$STATUS" = "RUNNING" ]; then
        # Check if cluster is accessible
        if gcloud container clusters get-credentials $NAME --location=$LOCATION &>/dev/null; then
            NODE_STATUS=$(kubectl get nodes --no-headers 2>/dev/null | grep -c "Ready" || echo "0")
            echo " (${NODE_STATUS} nodes ready)"
        else
            echo " (credentials not accessible)"
        fi
    else
        echo ""
    fi
done
```
Performs basic health checks on clusters.

### 9. Display cost-related information
```bash
echo -e "\n=== Cost Optimization Tips ==="
# Check for clusters that might be oversized
for cluster in $(gcloud container clusters list --format="value(name,location)"); do
    NAME=$(echo $cluster | awk '{print $1}')
    LOCATION=$(echo $cluster | awk '{print $2}')
    
    UTILIZATION=$(gcloud container clusters describe $NAME --location=$LOCATION --format="value(currentNodeCount)" 2>/dev/null)
    
    if [ "$UTILIZATION" = "0" ]; then
        echo "⚠️  $NAME has 0 nodes - consider deleting if unused"
    fi
done

# Check for old clusters
echo -e "\nClusters created over 30 days ago:"
THIRTY_DAYS_AGO=$(date -d "30 days ago" +%Y-%m-%d 2>/dev/null || date -v-30d +%Y-%m-%d)
gcloud container clusters list --format="table(name,createTime)" --filter="createTime<$THIRTY_DAYS_AGO" 2>/dev/null | tail -n +2
```
Identifies potential cost optimization opportunities.

## Validation
- Container API is enabled
- Cluster list displays correctly
- Status values are valid
- Location formats are correct

## Error Handling
- **"API not enabled"** - Enable with `gcloud services enable container.googleapis.com`
- **"Permission denied"** - Need container.clusters.list permission
- **"No clusters found"** - No GKE clusters exist in project
- **"Invalid location"** - Check region/zone format

## Safety Notes
- Read-only operation, safe to run frequently
- Getting credentials modifies local kubeconfig
- Large clusters may take time to describe
- Running clusters incur costs

## Examples
- **List all clusters**
  ```
  gcp-gke-clusters-list
  ```
  Shows all GKE clusters in the project

- **List clusters in specific region**
  ```
  gcp-gke-clusters-list "--region=us-central1"
  ```
  Shows only clusters in us-central1

- **List running clusters only**
  ```
  gcp-gke-clusters-list "--filter=status:RUNNING"
  ```
  Shows only operational clusters

- **List with custom format**
  ```
  gcp-gke-clusters-list "--format=json"
  ```
  Returns cluster data in JSON format