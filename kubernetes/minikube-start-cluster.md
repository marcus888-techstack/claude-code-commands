# Minikube Start Cluster

## Purpose
Start a local Kubernetes cluster using minikube for development and testing

## Context
Use minikube to run Kubernetes locally for development, testing, and learning. It creates a single-node or multi-node cluster in a VM or container, providing a full Kubernetes API without cloud infrastructure.

## Parameters
- `$DRIVER` - Virtualization driver to use
  - Optional
  - Default: Auto-detected
  - Options: `docker`, `virtualbox`, `hyperkit`, `kvm2`, `none`
- `$CPUS` - Number of CPUs to allocate
  - Optional
  - Default: `2`
  - Example: `4`, `8`
- `$MEMORY` - Amount of RAM to allocate
  - Optional
  - Default: `2048` (MB)
  - Example: `4096`, `8192`

## Steps

### 1. Basic cluster start
```bash
# Start with defaults
minikube start

# Start with specific resources
minikube start --cpus=$CPUS --memory=$MEMORY

# Start with specific driver
minikube start --driver=$DRIVER

# Start with Kubernetes version
minikube start --kubernetes-version=v1.28.0

# Start with custom profile name
minikube start -p dev-cluster
```
Starts basic minikube cluster.

### 2. Advanced configuration
```bash
# Start with multiple nodes
minikube start --nodes=3 --cpus=2 --memory=2048

# Start with specific container runtime
minikube start --container-runtime=containerd

# Start with custom DNS
minikube start --dns-domain=cluster.local

# Start with insecure registry
minikube start --insecure-registry="10.0.0.0/24"

# Start with feature gates
minikube start --feature-gates=EphemeralContainers=true

# Start with extra config
minikube start \
  --extra-config=kubelet.max-pods=110 \
  --extra-config=apiserver.enable-admission-plugins=PodSecurityPolicy
```
Advanced startup configurations.

### 3. Configure Docker environment
```bash
# Use minikube's Docker daemon
eval $(minikube docker-env)

# Verify Docker is pointing to minikube
docker ps
docker info | grep -i "server version"

# Build images directly in minikube
docker build -t myapp:local .

# Revert to host Docker
eval $(minikube docker-env -u)
```
Configures Docker integration.

### 4. Enable essential addons
```bash
# Enable ingress
minikube addons enable ingress

# Enable metrics server
minikube addons enable metrics-server

# Enable dashboard
minikube addons enable dashboard
minikube dashboard  # Opens in browser

# Enable registry
minikube addons enable registry

# List all addons
minikube addons list

# Enable multiple addons
for addon in ingress metrics-server dashboard; do
  minikube addons enable $addon
done
```
Enables common addons.

### 5. Configure networking
```bash
# Get minikube IP
minikube ip

# SSH into minikube
minikube ssh

# Create tunnel for LoadBalancer services
minikube tunnel

# Port forward to host
kubectl port-forward service/myapp 8080:80

# Access NodePort service
minikube service myapp --url

# Configure host DNS
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts
```
Sets up networking access.

### 6. Mount host directories
```bash
# Mount host directory
minikube mount $HOME/projects:/projects

# Mount with specific permissions
minikube mount $HOME/data:/data --uid=1000 --gid=1000

# Mount in background
nohup minikube mount $PWD:/app &

# Verify mount
minikube ssh "ls -la /app"
```
Shares files with cluster.

### 7. Resource and performance tuning
```bash
# Start with performance optimizations
minikube start \
  --cpus=4 \
  --memory=8192 \
  --disk-size=50g \
  --cache-images=true \
  --driver=docker

# Configure Docker resources (macOS)
minikube start \
  --driver=hyperkit \
  --hyperkit-vpnkit-sock=/var/run/vpnkit.sock

# Preload images
minikube image load nginx:latest
minikube image load redis:alpine

# Configure caching
minikube config set memory 4096
minikube config set cpus 4
minikube config set disk-size 50g
```
Optimizes performance.

### 8. Profile management
```bash
# Create development profile
minikube start -p development \
  --cpus=2 --memory=4096 \
  --kubernetes-version=v1.28.0

# Create staging profile
minikube start -p staging \
  --cpus=4 --memory=8192 \
  --kubernetes-version=v1.27.0

# List profiles
minikube profile list

# Switch profiles
minikube profile development

# Delete profile
minikube delete -p old-cluster

# Start with specific config
cat <<EOF > ~/.minikube/profiles/dev/config.json
{
  "cpus": 4,
  "memory": 8192,
  "driver": "docker",
  "kubernetes-version": "v1.28.0"
}
EOF
minikube start -p dev
```
Manages multiple clusters.

## Validation
- Cluster status is "Running"
- kubectl can connect to cluster
- All system pods are ready
- Docker daemon accessible (if configured)
- Addons are enabled and running

## Error Handling
- **"insufficient memory"** - Increase Docker/VM memory allocation
- **"driver not found"** - Install required virtualization software
- **"cluster already exists"** - Delete with `minikube delete` first
- **"connection refused"** - Check if cluster is running

## Safety Notes
- Minikube is for development only
- Data is not persistent by default
- Resource limits affect performance
- Some features differ from production
- Clean up unused clusters regularly

## Examples
- **Quick development start**
  ```bash
  minikube start --cpus=4 --memory=4096 --driver=docker
  minikube addons enable ingress metrics-server
  ```

- **Multi-node cluster**
  ```bash
  minikube start --nodes=3 --cpus=2 --memory=2048 -p multi-node
  ```

- **Full-featured setup**
  ```bash
  minikube start \
    --cpus=6 \
    --memory=12288 \
    --disk-size=100g \
    --addons=ingress,metrics-server,dashboard \
    --kubernetes-version=stable
  ```

- **Minimal resources**
  ```bash
  minikube start --cpus=1 --memory=1024 --disk-size=10g
  ```