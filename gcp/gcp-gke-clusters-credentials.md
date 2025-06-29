# GCP GKE Clusters Credentials

## Purpose
Fetch and configure kubectl credentials for accessing a GKE cluster

## Context
Use to set up local kubectl access to a GKE cluster. This configures your kubeconfig file with the necessary authentication and connection details. Required before running kubectl commands against the cluster.

## Parameters
- `$CLUSTER_NAME` - Name of the GKE cluster
  - Required
  - Example: `production-cluster`
- `$LOCATION` - Zone or region where cluster is located
  - Required
  - Example: `us-central1-a` or `us-central1`
- `$ARGUMENTS` - Additional configuration options
  - Optional
  - Example: `--internal-ip`

## Steps

### 1. Verify cluster exists
```bash
if ! gcloud container clusters describe $CLUSTER_NAME --location=$LOCATION &>/dev/null; then
    echo "Error: Cluster '$CLUSTER_NAME' not found in location '$LOCATION'"
    echo "Available clusters:"
    gcloud container clusters list --format="table(name,location)"
    exit 1
fi
```
Ensures the cluster exists before fetching credentials.

### 2. Display cluster information
```bash
echo "=== Cluster Information ==="
gcloud container clusters describe $CLUSTER_NAME --location=$LOCATION \
    --format="table(
        name,
        location,
        endpoint,
        masterVersion,
        currentNodeCount,
        status
    )"
```
Shows cluster details before connecting.

### 3. Check kubectl installation
```bash
echo -e "\n=== Checking Prerequisites ==="
if ! command -v kubectl &> /dev/null; then
    echo "⚠️  kubectl is not installed"
    echo "Install with: gcloud components install kubectl"
    exit 1
else
    echo "✓ kubectl is installed: $(kubectl version --client --short 2>/dev/null)"
fi
```
Ensures kubectl is available for cluster interaction.

### 4. Backup existing kubeconfig
```bash
if [ -f ~/.kube/config ]; then
    echo -e "\nBacking up existing kubeconfig..."
    cp ~/.kube/config ~/.kube/config.backup.$(date +%Y%m%d-%H%M%S)
    echo "✓ Backup saved"
fi
```
Preserves existing configurations before modification.

### 5. Fetch cluster credentials
```bash
echo -e "\n=== Fetching Credentials ==="
gcloud container clusters get-credentials $CLUSTER_NAME \
    --location=$LOCATION \
    $ARGUMENTS
```
Downloads and configures cluster access credentials.

### 6. Verify kubectl context
```bash
echo -e "\n=== Verifying Configuration ==="
CURRENT_CONTEXT=$(kubectl config current-context)
echo "Current context: $CURRENT_CONTEXT"

# Check if it matches our cluster
if [[ "$CURRENT_CONTEXT" == *"$CLUSTER_NAME"* ]]; then
    echo "✓ Context is set to the requested cluster"
else
    echo "⚠️  Context may not match requested cluster"
fi
```
Confirms kubectl is configured for the correct cluster.

### 7. Test cluster connectivity
```bash
echo -e "\n=== Testing Cluster Access ==="
echo -n "Connecting to cluster... "
if kubectl cluster-info &>/dev/null; then
    echo "✓ Success"
    
    # Show cluster endpoints
    echo -e "\nCluster endpoints:"
    kubectl cluster-info | grep -E "Kubernetes|DNS" | sed 's/^/  /'
else
    echo "✗ Failed"
    echo "Error: Unable to connect to cluster"
    exit 1
fi
```
Tests actual connectivity to the cluster.

### 8. Display cluster resources
```bash
echo -e "\n=== Cluster Resources ==="
echo "Nodes:"
kubectl get nodes --no-headers | wc -l | xargs echo "  Total:"
kubectl get nodes --no-headers | grep " Ready " | wc -l | xargs echo "  Ready:"

echo -e "\nNamespaces:"
kubectl get namespaces --no-headers | wc -l | xargs echo "  Total:"

echo -e "\nWorkloads:"
kubectl get deployments --all-namespaces --no-headers 2>/dev/null | wc -l | xargs echo "  Deployments:"
kubectl get pods --all-namespaces --no-headers 2>/dev/null | wc -l | xargs echo "  Pods:"
```
Shows basic cluster resource information.

### 9. Configure kubectl aliases (optional)
```bash
echo -e "\n=== Quick Access Setup ==="
ALIAS_NAME="k-${CLUSTER_NAME}"
echo "To create an alias for this cluster:"
echo "  alias $ALIAS_NAME='kubectl --context=$CURRENT_CONTEXT'"
echo ""
echo "To switch between clusters:"
echo "  kubectl config use-context <context-name>"
echo ""
echo "To list all contexts:"
echo "  kubectl config get-contexts"
```
Provides convenience commands for cluster access.

### 10. Display common kubectl commands
```bash
echo -e "\n=== Common Commands ==="
cat << 'EOF'
# View cluster info
kubectl cluster-info
kubectl get nodes

# Work with namespaces
kubectl get namespaces
kubectl get all -n <namespace>

# View workloads
kubectl get deployments --all-namespaces
kubectl get pods --all-namespaces
kubectl get services --all-namespaces

# Get pod logs
kubectl logs <pod-name> -n <namespace>

# Execute commands in pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash

# Apply configurations
kubectl apply -f <yaml-file>

# Delete resources
kubectl delete -f <yaml-file>
EOF
```
Provides reference for common operations.

## Validation
- Cluster exists and is accessible
- kubectl context is configured correctly
- Can connect to cluster API
- Basic kubectl commands work

## Error Handling
- **"Cluster not found"** - Verify cluster name and location
- **"Permission denied"** - Need container.clusters.get permission
- **"kubectl not found"** - Install kubectl first
- **"Connection refused"** - Check firewall rules and network
- **"Unauthorized"** - Re-authenticate with gcloud

## Safety Notes
- Modifies ~/.kube/config file
- Overwrites context if it already exists
- Backup is created automatically
- Credentials expire and need periodic refresh
- Be cautious with production clusters

## Examples
- **Get credentials for zonal cluster**
  ```
  gcp-gke-clusters-credentials my-cluster us-central1-a
  ```
  Configures access to zonal cluster

- **Get credentials for regional cluster**
  ```
  gcp-gke-clusters-credentials prod-cluster us-central1
  ```
  Configures access to regional cluster

- **Use internal IP only**
  ```
  gcp-gke-clusters-credentials private-cluster us-east1-b "--internal-ip"
  ```
  Connects using internal IP for private clusters