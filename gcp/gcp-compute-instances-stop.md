# GCP Compute Instances Stop

## Purpose
Stop one or more running Compute Engine VM instances to save costs

## Context
Use to stop instances that don't need to run continuously. Stopped instances don't incur compute charges but still incur storage charges for attached disks. Useful for development environments, testing, or scheduled workloads.

## Parameters
- `$INSTANCE_NAME` - Name of the instance(s) to stop
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
    if [ "$STATUS" == "TERMINATED" ]; then
        echo "Warning: Instance '$instance' is already stopped"
    fi
done
```
Checks that instances exist and shows current status.

### 2. Display instance details before stopping
```bash
echo -e "\n=== Instance Details ==="
gcloud compute instances list --filter="name:($INSTANCE_NAME)" --zones=$ZONE \
    --format="table(name,machineType.machine_type(),status,externalIp)"
```
Shows instance configuration and IPs before stopping.

### 3. Warn about running processes
```bash
if [[ ! " $* " =~ " --force " ]]; then
    echo -e "\n⚠️  WARNING: Stopping instances will terminate all running processes"
    read -p "Continue? (y/N): " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        echo "Operation cancelled"
        exit 1
    fi
fi
```
Warns user about data loss and confirms action.

### 4. Stop the instances
```bash
echo -e "\nStopping instance(s)..."
gcloud compute instances stop $INSTANCE_NAME --zone=$ZONE
```
Initiates the stop operation for specified instances.

### 5. Monitor shutdown progress
```bash
echo -e "\n=== Shutdown Progress ==="
for i in {1..60}; do
    ALL_STOPPED=true
    for instance in $INSTANCE_NAME; do
        STATUS=$(gcloud compute instances describe $instance --zone=$ZONE --format="value(status)" 2>/dev/null)
        if [ "$STATUS" != "TERMINATED" ]; then
            ALL_STOPPED=false
            echo -ne "\r$instance: $STATUS... (${i}s)"
        fi
    done
    
    if [ "$ALL_STOPPED" = true ]; then
        echo -e "\n✓ All instances are stopped"
        break
    fi
    
    if [ $i -eq 60 ]; then
        echo -e "\n⚠️  Timeout: Some instances may still be stopping"
    fi
    sleep 1
done
```
Monitors instances until they reach TERMINATED state.

### 6. Show final status
```bash
echo -e "\n=== Final Status ==="
gcloud compute instances list --filter="name:($INSTANCE_NAME)" --zones=$ZONE \
    --format="table(name,status,lastStopTimestamp)"
```
Confirms all instances are stopped with timestamps.

### 7. Display cost savings information
```bash
echo -e "\n=== Cost Information ==="
for instance in $INSTANCE_NAME; do
    MACHINE_TYPE=$(gcloud compute instances describe $instance --zone=$ZONE --format="value(machineType.scope(machineTypes).machineType)")
    echo "$instance ($MACHINE_TYPE):"
    echo "  ✓ Compute charges: STOPPED"
    echo "  ⚠️  Storage charges: CONTINUE for attached disks"
done
echo -e "\nNote: To completely avoid charges, delete instances with:"
echo "gcloud compute instances delete $INSTANCE_NAME --zone=$ZONE"
```
Informs about cost implications of stopped instances.

### 8. Show restart instructions
```bash
echo -e "\n=== Restart Instructions ==="
echo "To restart these instances later:"
echo "  gcp-compute-instances-start \"$INSTANCE_NAME\" $ZONE"
```
Provides command to restart instances when needed.

## Validation
- All specified instances reach TERMINATED state
- Stop timestamps are recorded
- Instances no longer appear in running list
- No active SSH connections remain

## Error Handling
- **"Instance not found"** - Check instance name and zone
- **"Permission denied"** - Need compute.instances.stop permission
- **"Instance is already stopped"** - No action needed
- **"Operation timed out"** - Instance may have issues shutting down
- **"Cannot stop instance"** - Check if instance has special configurations

## Safety Notes
- All running processes will be terminated
- Unsaved data in memory will be lost
- Attached persistent disks are preserved
- Static IPs are retained but may incur charges
- Consider graceful shutdown of applications first

## Examples
- **Stop single instance**
  ```
  gcp-compute-instances-stop web-server-1 us-central1-a
  ```
  Stops web-server-1 with confirmation

- **Stop multiple instances**
  ```
  gcp-compute-instances-stop "app-1 app-2 app-3" us-east1-b
  ```
  Stops multiple instances in the same zone

- **Force stop without confirmation**
  ```
  gcp-compute-instances-stop test-instance us-west1-a --force
  ```
  Stops instance without confirmation prompt