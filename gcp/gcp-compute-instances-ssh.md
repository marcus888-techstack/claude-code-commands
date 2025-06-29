# GCP Compute Instances SSH

## Purpose
Connect to a Compute Engine VM instance via SSH using Google Cloud's IAP (Identity-Aware Proxy)

## Context
Use to securely access VM instances without exposing SSH ports to the internet. Uses Google's Identity-Aware Proxy for secure tunneling. Automatically handles SSH key generation and management.

## Parameters
- `$INSTANCE_NAME` - Name of the instance to connect to
  - Required
  - Example: `web-server-1`
- `$ZONE` - Zone where the instance is located
  - Required  
  - Example: `us-central1-a`
- `$ARGUMENTS` - Additional SSH arguments or commands
  - Optional
  - Example: `--command="ls -la"` or `-- -L 8080:localhost:80`

## Steps

### 1. Verify instance exists and is running
```bash
STATUS=$(gcloud compute instances describe $INSTANCE_NAME --zone=$ZONE --format="value(status)" 2>/dev/null)
if [ -z "$STATUS" ]; then
    echo "Error: Instance '$INSTANCE_NAME' not found in zone '$ZONE'"
    exit 1
fi

if [ "$STATUS" != "RUNNING" ]; then
    echo "Error: Instance is $STATUS (must be RUNNING)"
    echo "Start it with: gcp-compute-instances-start $INSTANCE_NAME $ZONE"
    exit 1
fi
```
Ensures instance exists and is in a connectable state.

### 2. Check SSH key configuration
```bash
echo "Checking SSH keys..."
if [ ! -f ~/.ssh/google_compute_engine ]; then
    echo "Google Compute SSH key not found. It will be created on first connection."
fi
```
Verifies SSH keys exist or will be created.

### 3. Display instance connection details
```bash
echo -e "\n=== Instance Details ==="
gcloud compute instances describe $INSTANCE_NAME --zone=$ZONE \
    --format="table(name,status,networkInterfaces[0].networkIP,networkInterfaces[0].accessConfigs[0].natIP)"
```
Shows instance network configuration.

### 4. Check IAP tunnel authorization
```bash
echo -e "\nChecking IAP tunnel access..."
if gcloud compute ssh $INSTANCE_NAME --zone=$ZONE --tunnel-through-iap --dry-run 2>&1 | grep -q "Permission denied"; then
    echo "⚠️  You may not have IAP tunnel permissions"
    echo "Ask admin to grant: roles/iap.tunnelResourceAccessor"
fi
```
Verifies IAP tunnel permissions are configured.

### 5. Establish SSH connection
```bash
echo -e "\nConnecting to $INSTANCE_NAME..."
if [ -z "$ARGUMENTS" ]; then
    # Interactive SSH session
    gcloud compute ssh $INSTANCE_NAME --zone=$ZONE --tunnel-through-iap
else
    # SSH with additional arguments or command
    gcloud compute ssh $INSTANCE_NAME --zone=$ZONE --tunnel-through-iap $ARGUMENTS
fi
```
Connects to the instance using SSH through IAP.

### 6. Handle connection errors
```bash
if [ $? -ne 0 ]; then
    echo -e "\n=== Troubleshooting ==="
    echo "1. Check if instance allows SSH:"
    echo "   gcloud compute firewall-rules list --filter='allowed[].ports:(22)'"
    echo ""
    echo "2. Check OS Login status:"
    echo "   gcloud compute project-info describe --format='value(commonInstanceMetadata.items[enable-oslogin])'"
    echo ""
    echo "3. Try direct SSH (requires external IP):"
    echo "   gcloud compute ssh $INSTANCE_NAME --zone=$ZONE"
    echo ""
    echo "4. Check instance serial console:"
    echo "   gcloud compute instances get-serial-port-output $INSTANCE_NAME --zone=$ZONE"
fi
```
Provides troubleshooting steps if connection fails.

## Validation
- Instance is in RUNNING state
- SSH connection is established
- Commands execute on remote instance
- Interactive session works properly

## Error Handling
- **"Permission denied (publickey)"** - SSH keys need to be configured
- **"Could not fetch resource"** - Instance doesn't exist or wrong zone
- **"Instance not running"** - Start the instance first
- **"IAP tunnel failed"** - Check IAP permissions and firewall rules
- **"Connection timed out"** - Instance may not be fully booted

## Safety Notes
- SSH keys are automatically managed by gcloud
- IAP tunnel provides secure access without public IPs
- All SSH sessions are logged for audit
- Avoid running destructive commands without confirmation

## Examples
- **Interactive SSH session**
  ```
  gcp-compute-instances-ssh web-server-1 us-central1-a
  ```
  Opens interactive shell on the instance

- **Run single command**
  ```
  gcp-compute-instances-ssh app-server us-east1-b "--command='df -h'"
  ```
  Executes command and returns output

- **Port forwarding**
  ```
  gcp-compute-instances-ssh database-1 us-west1-a "-- -L 5432:localhost:5432"
  ```
  Forwards local port 5432 to instance's PostgreSQL

- **Copy files**
  ```
  gcloud compute scp file.txt $INSTANCE_NAME:~/file.txt --zone=$ZONE
  ```
  Copies files to/from instance (separate command)