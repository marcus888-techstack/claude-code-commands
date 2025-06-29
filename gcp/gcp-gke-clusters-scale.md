# GCP GKE Clusters Scale

## Purpose
Scale the node count of a GKE cluster node pool up or down

## Context
Use to adjust cluster capacity by changing the number of nodes. Useful for handling varying workloads, reducing costs during low usage, or adding capacity for increased demand. Affects billing immediately.

## Parameters
- `$CLUSTER_NAME` - Name of the GKE cluster
  - Required
  - Example: `production-cluster`
- `$NODE_COUNT` - Target number of nodes
  - Required
  - Example: `3`, `10`
- `$LOCATION` - Zone or region where cluster is located
  - Required
  - Example: `us-central1-a`
- `$NODE_POOL` - Name of node pool to scale
  - Optional
  - Default: `default-pool`

## Steps

### 1. Verify cluster and node pool exist
```bash
if ! gcloud container clusters describe $CLUSTER_NAME --location=$LOCATION &>/dev/null; then
    echo "Error: Cluster '$CLUSTER_NAME' not found in location '$LOCATION'"
    exit 1
fi

NODE_POOL=${NODE_POOL:-default-pool}
if ! gcloud container node-pools describe $NODE_POOL --cluster=$CLUSTER_NAME --location=$LOCATION &>/dev/null; then
    echo "Error: Node pool '$NODE_POOL' not found"
    echo "Available node pools:"
    gcloud container node-pools list --cluster=$CLUSTER_NAME --location=$LOCATION --format="table(name)"
    exit 1
fi
```
Ensures cluster and node pool exist.

### 2. Validate node count
```bash
if ! [[ "$NODE_COUNT" =~ ^[0-9]+$ ]] || [ "$NODE_COUNT" -lt 0 ]; then
    echo "Error: Node count must be a non-negative integer"
    exit 1
fi

# Warn about scaling to zero
if [ "$NODE_COUNT" -eq 0 ]; then
    echo "⚠️  WARNING: Scaling to 0 nodes will stop all workloads!"
    read -p "Are you sure? (yes/N): " -r
    if [[ ! $REPLY =~ ^yes$ ]]; then
        echo "Scaling cancelled"
        exit 1
    fi
fi
```
Validates the requested node count.

### 3. Display current state
```bash
echo "=== Current Cluster State ==="
CURRENT_INFO=$(gcloud container node-pools describe $NODE_POOL \
    --cluster=$CLUSTER_NAME \
    --location=$LOCATION \
    --format="value(initialNodeCount,autoscaling.minNodeCount,autoscaling.maxNodeCount)")

CURRENT_COUNT=$(gcloud container clusters describe $CLUSTER_NAME \
    --location=$LOCATION \
    --format="value(currentNodeCount)")

echo "Cluster: $CLUSTER_NAME"
echo "Node Pool: $NODE_POOL"
echo "Current nodes: $CURRENT_COUNT"
echo "Target nodes: $NODE_COUNT"

# Check if autoscaling is enabled
if echo "$CURRENT_INFO" | grep -q "[0-9]"; then
    MIN_NODES=$(echo "$CURRENT_INFO" | awk '{print $2}')
    MAX_NODES=$(echo "$CURRENT_INFO" | awk '{print $3}')
    if [ -n "$MIN_NODES" ] && [ -n "$MAX_NODES" ]; then
        echo "⚠️  Autoscaling is enabled (min: $MIN_NODES, max: $MAX_NODES)"
        echo "Manual scaling will be overridden by autoscaler"
    fi
fi
```
Shows current configuration before scaling.

### 4. Check workload impact
```bash
if command -v kubectl &> /dev/null; then
    echo -e "\n=== Workload Impact Analysis ==="
    gcloud container clusters get-credentials $CLUSTER_NAME --location=$LOCATION &>/dev/null
    
    TOTAL_PODS=$(kubectl get pods --all-namespaces --no-headers 2>/dev/null | wc -l)
    NODES_AVAILABLE=$(kubectl get nodes --no-headers 2>/dev/null | grep " Ready " | wc -l)
    
    if [ "$NODES_AVAILABLE" -gt 0 ]; then
        PODS_PER_NODE=$((TOTAL_PODS / NODES_AVAILABLE))
        echo "Current pods: $TOTAL_PODS"
        echo "Average pods per node: $PODS_PER_NODE"
        
        if [ "$NODE_COUNT" -lt "$CURRENT_COUNT" ]; then
            echo "⚠️  Scaling down may cause pod evictions"
        fi
    fi
fi
```
Analyzes potential impact on running workloads.

### 5. Perform scaling operation
```bash
echo -e "\n=== Scaling Cluster ==="
echo "Scaling from $CURRENT_COUNT to $NODE_COUNT nodes..."

gcloud container clusters resize $CLUSTER_NAME \
    --node-pool=$NODE_POOL \
    --num-nodes=$NODE_COUNT \
    --location=$LOCATION \
    --quiet
```
Executes the scaling operation.

### 6. Monitor scaling progress
```bash
echo -e "\n=== Monitoring Progress ==="
echo "Waiting for scaling to complete..."

for i in {1..30}; do
    NEW_COUNT=$(gcloud container clusters describe $CLUSTER_NAME \
        --location=$LOCATION \
        --format="value(currentNodeCount)" 2>/dev/null)
    
    echo -ne "\rCurrent nodes: $NEW_COUNT (target: $NODE_COUNT)... "
    
    if [ "$NEW_COUNT" -eq "$NODE_COUNT" ]; then
        echo "✓ Complete"
        break
    fi
    
    if [ $i -eq 30 ]; then
        echo "⚠️  Timeout - scaling may still be in progress"
    fi
    
    sleep 10
done
```
Monitors the scaling operation progress.

### 7. Verify node health
```bash
if command -v kubectl &> /dev/null; then
    echo -e "\n=== Node Health Check ==="
    gcloud container clusters get-credentials $CLUSTER_NAME --location=$LOCATION &>/dev/null
    
    echo "Node status:"
    kubectl get nodes --no-headers | awk '{print "  " $1 ": " $2}'
    
    NOT_READY=$(kubectl get nodes --no-headers | grep -v " Ready " | wc -l)
    if [ "$NOT_READY" -gt 0 ]; then
        echo "⚠️  $NOT_READY node(s) are not ready"
    else
        echo "✓ All nodes are ready"
    fi
fi
```
Checks health of nodes after scaling.

### 8. Display cost implications
```bash
echo -e "\n=== Cost Implications ==="
MACHINE_TYPE=$(gcloud container node-pools describe $NODE_POOL \
    --cluster=$CLUSTER_NAME \
    --location=$LOCATION \
    --format="value(config.machineType)")

echo "Machine type: $MACHINE_TYPE"
echo "Node count change: $CURRENT_COUNT → $NODE_COUNT"

if [ "$NODE_COUNT" -gt "$CURRENT_COUNT" ]; then
    ADDED=$((NODE_COUNT - CURRENT_COUNT))
    echo "📈 Added $ADDED node(s) - costs will increase"
elif [ "$NODE_COUNT" -lt "$CURRENT_COUNT" ]; then
    REMOVED=$((CURRENT_COUNT - NODE_COUNT))
    echo "📉 Removed $REMOVED node(s) - costs will decrease"
else
    echo "No change in node count"
fi

echo -e "\nNote: Billing changes take effect immediately"
```
Shows cost impact of scaling operation.

## Validation
- Cluster reaches target node count
- All nodes are in Ready state
- Workloads are redistributed properly
- No unexpected pod evictions

## Error Handling
- **"Cluster not found"** - Verify cluster name and location
- **"Permission denied"** - Need container.clusters.update permission
- **"Invalid node count"** - Must be non-negative integer
- **"Quota exceeded"** - Check compute quotas for region
- **"Insufficient resources"** - Region may lack capacity

## Safety Notes
- Scaling down may cause pod evictions
- Scaling to zero stops all workloads
- Costs change immediately with node count
- Consider autoscaling instead of manual scaling
- Monitor workload performance after scaling

## Examples
- **Scale up cluster**
  ```
  gcp-gke-clusters-scale production-cluster 10 us-central1-a
  ```
  Scales cluster to 10 nodes

- **Scale down cluster**
  ```
  gcp-gke-clusters-scale dev-cluster 2 us-east1-b
  ```
  Scales cluster down to 2 nodes

- **Scale specific node pool**
  ```
  gcp-gke-clusters-scale my-cluster 5 us-west1-a high-mem-pool
  ```
  Scales high-mem-pool to 5 nodes

- **Scale to zero (careful!)**
  ```
  gcp-gke-clusters-scale test-cluster 0 us-central1-c
  ```
  Scales down to zero nodes