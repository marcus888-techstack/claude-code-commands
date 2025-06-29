# GCP Compute Instances List

## Purpose
List all Compute Engine VM instances across zones with their status and details

## Context
Use to get an overview of all virtual machines in your project. Shows instance names, zones, machine types, and current status. Essential for managing VM infrastructure and identifying running resources.

## Parameters
- `$ARGUMENTS` - Optional filters or format options
  - Optional
  - Example: `--filter=status:RUNNING`, `--zones=us-central1-a`

## Steps

### 1. Check Compute API status
```bash
if ! gcloud services list --enabled --filter="name:compute.googleapis.com" --format="value(name)" | grep -q compute; then
    echo "Warning: Compute Engine API is not enabled"
    echo "Enable it with: gcloud services enable compute.googleapis.com"
    exit 1
fi
```
Ensures the Compute Engine API is enabled.

### 2. Display project context
```bash
echo "Project: $(gcloud config get-value project)"
echo "Listing all compute instances..."
echo "========================================"
```
Shows which project's instances are being listed.

### 3. List all instances across zones
```bash
gcloud compute instances list $ARGUMENTS
```
Displays all instances with name, zone, machine type, internal/external IPs, and status.

### 4. Show instance statistics
```bash
echo "========================================"
TOTAL=$(gcloud compute instances list --format="value(name)" | wc -l)
RUNNING=$(gcloud compute instances list --filter="status:RUNNING" --format="value(name)" | wc -l)
STOPPED=$(gcloud compute instances list --filter="status:TERMINATED" --format="value(name)" | wc -l)

echo "Total instances: $TOTAL"
echo "Running: $RUNNING"
echo "Stopped: $STOPPED"
```
Provides summary counts by status.

### 5. Group by zone
```bash
echo -e "\n=== Instances by Zone ==="
for zone in $(gcloud compute instances list --format="value(zone)" | sort -u); do
    COUNT=$(gcloud compute instances list --filter="zone:$zone" --format="value(name)" | wc -l)
    echo "$zone: $COUNT instance(s)"
done
```
Shows distribution of instances across zones.

### 6. Show machine type distribution
```bash
echo -e "\n=== Machine Type Distribution ==="
gcloud compute instances list --format="value(machineType.scope(machineTypes))" | \
    sed 's/.*\///' | sort | uniq -c | sort -rn | head -10
```
Displays the most common machine types in use.

### 7. Calculate resource usage
```bash
echo -e "\n=== Resource Summary ==="
TOTAL_CPUS=$(gcloud compute instances list --filter="status:RUNNING" \
    --format="table(name,machineType.machine_type().guestCpus)" | \
    tail -n +2 | awk '{sum += $2} END {print sum}')
echo "Total running vCPUs: ${TOTAL_CPUS:-0}"
```
Shows aggregate resource consumption.

### 8. List instances with labels
```bash
echo -e "\n=== Labeled Instances ==="
gcloud compute instances list --format="table(name,labels.list())" --filter="labels:*" | head -10
```
Shows instances with labels for organization.

## Validation
- Compute API is enabled
- Instance list displays correctly
- Status values are valid (RUNNING, TERMINATED, etc.)
- Zone names are properly formatted

## Error Handling
- **"API not enabled"** - Enable with `gcloud services enable compute.googleapis.com`
- **"Permission denied"** - Need compute.instances.list permission
- **"Listed 0 items"** - No instances exist in the project
- **"Invalid filter"** - Check filter syntax

## Safety Notes
- Read-only operation, safe to run frequently
- May take time with many instances
- Costs may apply for running instances shown

## Examples
- **List all instances**
  ```
  gcp-compute-instances-list
  ```
  Shows all instances in all zones

- **List running instances only**
  ```
  gcp-compute-instances-list "--filter=status:RUNNING"
  ```
  Shows only instances that are currently running

- **List instances in specific zone**
  ```
  gcp-compute-instances-list "--zones=us-central1-a"
  ```
  Shows instances in us-central1-a zone only

- **List with custom format**
  ```
  gcp-compute-instances-list "--format=table(name,status,machineType.machine_type())"
  ```
  Shows specific fields in table format