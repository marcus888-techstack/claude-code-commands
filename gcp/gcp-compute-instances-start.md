# GCP Compute Instances Start

## Purpose
Start one or more stopped Compute Engine VM instances

## Context
Use to start instances that are currently stopped (TERMINATED state). Starting instances will resume billing for compute resources. Useful for development environments or scheduled workloads that don't need to run continuously.

## Parameters
- `$INSTANCE_NAME` - Name of the instance(s) to start
  - Required
  - Example: `web-server-1` or `web-server-1 web-server-2`
- `$ZONE` - Zone where the instance is located
  - Required
  - Example: `us-central1-a`

## Steps

### 1. Verify instance exists and status
```bash
echo "Checking instance status..."
for instance in $INSTANCE_NAME; do
    STATUS=$(gcloud compute instances describe $instance --zone=$ZONE --format="value(status)" 2>/dev/null)
    if [ -z "$STATUS" ]; then
        echo "Error: Instance '$instance' not found in zone '$ZONE'"
        exit 1
    fi
    echo "$instance: $STATUS"
    if [ "$STATUS" == "RUNNING" ]; then
        echo "Warning: Instance '$instance' is already running"
    fi
done
```
Checks that instances exist and shows current status.

### 2. Display instance details before starting
```bash
echo -e "\n=== Instance Details ==="
gcloud compute instances list --filter="name:($INSTANCE_NAME)" --zones=$ZONE \
    --format="table(name,machineType.machine_type(),status,externalIp)"
```
Shows instance configuration before starting.

### 3. Start the instances
```bash
echo -e "\nStarting instance(s)..."
gcloud compute instances start $INSTANCE_NAME --zone=$ZONE
```
Initiates the start operation for the specified instances.

### 4. Monitor startup progress
```bash
echo -e "\n=== Startup Progress ==="
for i in {1..30}; do
    ALL_RUNNING=true
    for instance in $INSTANCE_NAME; do
        STATUS=$(gcloud compute instances describe $instance --zone=$ZONE --format="value(status)" 2>/dev/null)
        if [ "$STATUS" != "RUNNING" ]; then
            ALL_RUNNING=false
            echo -ne "\r$instance: $STATUS... (${i}s)"
        fi
    done
    
    if [ "$ALL_RUNNING" = true ]; then
        echo -e "\n✓ All instances are running"
        break
    fi
    
    if [ $i -eq 30 ]; then
        echo -e "\n⚠️  Timeout: Some instances may still be starting"
    fi
    sleep 1
done
```
Monitors instances until they reach RUNNING state.

### 5. Verify instances are accessible
```bash
echo -e "\n=== Instance Status ==="
for instance in $INSTANCE_NAME; do
    EXTERNAL_IP=$(gcloud compute instances describe $instance --zone=$ZONE --format="value(networkInterfaces[0].accessConfigs[0].natIP)")
    INTERNAL_IP=$(gcloud compute instances describe $instance --zone=$ZONE --format="value(networkInterfaces[0].networkIP)")
    echo "$instance:"
    echo "  Status: RUNNING"
    echo "  Internal IP: $INTERNAL_IP"
    echo "  External IP: ${EXTERNAL_IP:-None}"
done
```
Shows IP addresses for connecting to the instances.

### 6. Check system startup
```bash
echo -e "\n=== Checking System Startup ==="
for instance in $INSTANCE_NAME; do
    echo -n "$instance: "
    if gcloud compute ssh $instance --zone=$ZONE --command="echo 'SSH ready'" 2>/dev/null; then
        echo "✓ SSH is ready"
    else
        echo "⚠️  SSH not ready yet (system may still be booting)"
    fi
done
```
Tests SSH connectivity to verify full system startup.

### 7. Display connection instructions
```bash
echo -e "\n=== Connection Instructions ==="
for instance in $INSTANCE_NAME; do
    echo "SSH to $instance:"
    echo "  gcloud compute ssh $instance --zone=$ZONE"
done
```
Provides commands to connect to the started instances.

## Validation
- All specified instances reach RUNNING state
- External/Internal IPs are assigned
- Instances appear in running instances list
- SSH connectivity is established (when ready)

## Error Handling
- **"Instance not found"** - Check instance name and zone
- **"Permission denied"** - Need compute.instances.start permission
- **"Quota exceeded"** - Check CPU/IP quotas for the region
- **"Instance is already running"** - No action needed
- **"Zone not found"** - Verify zone name is correct

## Safety Notes
- Starting instances resumes billing immediately
- Ensure instances have proper security groups configured
- Some instances may take several minutes to fully boot
- System services may need additional time after instance starts

## Examples
- **Start single instance**
  ```
  gcp-compute-instances-start web-server-1 us-central1-a
  ```
  Starts web-server-1 in us-central1-a zone

- **Start multiple instances**
  ```
  gcp-compute-instances-start "web-1 web-2 web-3" us-east1-b
  ```
  Starts multiple instances in the same zone

- **Start development environment**
  ```
  gcp-compute-instances-start dev-instance us-west1-a
  ```
  Starts a development instance