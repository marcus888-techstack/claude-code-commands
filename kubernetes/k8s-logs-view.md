# K8s Logs View

## Purpose
View and analyze logs from Kubernetes pods, deployments, and other resources for debugging and monitoring

## Context
Use to troubleshoot application issues, monitor behavior, and understand what's happening inside containers. Supports viewing logs from multiple containers, following logs in real-time, and filtering by time or patterns.

## Parameters
- `$RESOURCE` - Resource type and name
  - Required
  - Example: `pod/frontend-abc123`, `deployment/api-server`
- `$CONTAINER` - Specific container name
  - Optional
  - Default: First container
  - Example: `app`, `sidecar`, `init-container`
- `$NAMESPACE` - Kubernetes namespace
  - Optional
  - Default: `default`

## Steps

### 1. View basic pod logs
```bash
# Current logs from pod
kubectl logs $POD_NAME -n $NAMESPACE

# Logs from specific container
kubectl logs $POD_NAME -c $CONTAINER -n $NAMESPACE

# Previous container logs (if crashed)
kubectl logs $POD_NAME -n $NAMESPACE --previous

# All containers in pod
kubectl logs $POD_NAME -n $NAMESPACE --all-containers=true
```
Basic log viewing commands.

### 2. Follow logs in real-time
```bash
# Follow log output
kubectl logs -f $POD_NAME -n $NAMESPACE

# Follow with timestamps
kubectl logs -f $POD_NAME -n $NAMESPACE --timestamps

# Follow with line prefix
kubectl logs -f $POD_NAME -n $NAMESPACE --prefix=true

# Follow multiple pods
kubectl logs -f -l app=myapp -n $NAMESPACE --all-containers=true
```
Real-time log monitoring.

### 3. Filter logs by time
```bash
# Logs from last hour
kubectl logs $POD_NAME -n $NAMESPACE --since=1h

# Logs from last 15 minutes
kubectl logs $POD_NAME -n $NAMESPACE --since=15m

# Logs since specific time
kubectl logs $POD_NAME -n $NAMESPACE --since-time=2024-01-15T10:00:00Z

# Last N lines
kubectl logs $POD_NAME -n $NAMESPACE --tail=100

# Combine time and line filters
kubectl logs $POD_NAME -n $NAMESPACE --since=30m --tail=50
```
Time-based log filtering.

### 4. View logs from deployments
```bash
# Logs from all pods in deployment
kubectl logs deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Follow deployment logs
kubectl logs -f deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Logs from replica set
kubectl logs rs/$REPLICASET_NAME -n $NAMESPACE

# Logs from statefulset
kubectl logs statefulset/$STATEFULSET_NAME -n $NAMESPACE
```
Logs from higher-level resources.

### 5. Advanced log filtering
```bash
# Logs from pods with label
kubectl logs -l app=frontend -n $NAMESPACE

# Logs from multiple labels
kubectl logs -l app=backend,version=v2 -n $NAMESPACE

# Grep specific patterns
kubectl logs $POD_NAME -n $NAMESPACE | grep ERROR

# Exclude patterns
kubectl logs $POD_NAME -n $NAMESPACE | grep -v DEBUG

# Complex filtering with awk
kubectl logs $POD_NAME -n $NAMESPACE | awk '/ERROR/ {print $0}'
```
Pattern-based filtering.

### 6. Export and analyze logs
```bash
# Save logs to file
kubectl logs $POD_NAME -n $NAMESPACE > pod-logs.txt

# Save with timestamps
kubectl logs $POD_NAME -n $NAMESPACE --timestamps > pod-logs-$(date +%Y%m%d-%H%M%S).txt

# Compress large logs
kubectl logs $POD_NAME -n $NAMESPACE | gzip > pod-logs.gz

# JSON logs pretty print
kubectl logs $POD_NAME -n $NAMESPACE | jq '.'

# Count log levels
kubectl logs $POD_NAME -n $NAMESPACE | grep -E 'ERROR|WARN|INFO' | sort | uniq -c
```
Log export and analysis.

### 7. Multi-container and init containers
```bash
# List all containers in pod
kubectl get pod $POD_NAME -n $NAMESPACE \
  -o jsonpath='{.spec.containers[*].name}'

# Logs from init containers
kubectl logs $POD_NAME -c $INIT_CONTAINER_NAME -n $NAMESPACE

# All containers including init
kubectl logs $POD_NAME -n $NAMESPACE --all-containers=true --prefix=true

# Specific container patterns
for container in $(kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.spec.containers[*].name}'); do
  echo "=== Logs from $container ==="
  kubectl logs $POD_NAME -c $container -n $NAMESPACE --tail=20
done
```
Multi-container log handling.

### 8. Troubleshooting with logs
```bash
# Check why pod is crashing
kubectl logs $POD_NAME -n $NAMESPACE --previous

# Combined with events
kubectl describe pod $POD_NAME -n $NAMESPACE | tail -20

# Watch logs during startup
kubectl logs -f $POD_NAME -n $NAMESPACE --since=1s

# Debug CrashLoopBackOff
kubectl logs $POD_NAME -n $NAMESPACE --previous --tail=50

# Check all failing pods
kubectl get pods -n $NAMESPACE | grep -E 'Error|CrashLoopBackOff' | \
  awk '{print $1}' | xargs -I {} kubectl logs {} -n $NAMESPACE --tail=10

# Aggregate logs from multiple pods
kubectl logs -l app=myapp -n $NAMESPACE --all-containers=true \
  --prefix=true --timestamps | sort
```
Common troubleshooting patterns.

## Validation
- Logs are retrieved successfully
- Timestamps are visible (if requested)
- Container names are correct
- No permission errors
- Log output is readable

## Error Handling
- **"container not found"** - List containers first
- **"pod not found"** - Check pod name and namespace
- **"previous terminated container not found"** - Container hasn't crashed yet
- **"forbidden"** - Check RBAC permissions for pods/logs

## Safety Notes
- Logs may contain sensitive information
- Large logs can consume bandwidth
- Use `--tail` to limit output size
- Be careful with log aggregation in production
- Consider log retention policies

## Examples
- **Debug crashing pod**
  ```bash
  kubectl logs crashpod-abc123 --previous --tail=100
  ```

- **Follow application logs**
  ```bash
  kubectl logs -f deployment/webapp --all-containers=true
  ```

- **Export error logs**
  ```bash
  kubectl logs pod/api-123 --since=1h | grep ERROR > errors.log
  ```

- **Monitor multiple pods**
  ```bash
  kubectl logs -f -l app=frontend,env=prod --prefix=true --timestamps
  ```