# K8s Pod Debug

## Purpose
Debug a problematic pod by examining logs, describing resources, and executing commands inside containers

## Context
Use when a pod is not running as expected, showing errors, or experiencing issues. This command helps diagnose problems by providing multiple debugging approaches including logs, events, and interactive shell access.

## Parameters
- `$POD_NAME` - Name of the pod to debug
  - Required
  - Example: `frontend-5d8c7b9f-x2n4k`
- `$NAMESPACE` - Kubernetes namespace
  - Optional
  - Default: `default`
  - Example: `production`, `staging`
- `$CONTAINER` - Specific container in multi-container pods
  - Optional
  - Default: First container
  - Example: `app`, `sidecar`

## Steps

### 1. Get pod status and events
```bash
# Basic pod information
kubectl get pod $POD_NAME -n $NAMESPACE

# Detailed pod description with events
kubectl describe pod $POD_NAME -n $NAMESPACE

# Get pod events separately
kubectl get events --field-selector involvedObject.name=$POD_NAME -n $NAMESPACE
```
Shows current status and recent events.

### 2. Check pod logs
```bash
# Current logs
kubectl logs $POD_NAME -n $NAMESPACE -c $CONTAINER

# Previous container logs (if crashed)
kubectl logs $POD_NAME -n $NAMESPACE -c $CONTAINER --previous

# Follow logs in real-time
kubectl logs -f $POD_NAME -n $NAMESPACE -c $CONTAINER

# Logs with timestamps
kubectl logs $POD_NAME -n $NAMESPACE -c $CONTAINER --timestamps

# Last N lines
kubectl logs $POD_NAME -n $NAMESPACE -c $CONTAINER --tail=100
```
Retrieves container logs for error analysis.

### 3. Execute commands in pod
```bash
# Open interactive shell
kubectl exec -it $POD_NAME -n $NAMESPACE -c $CONTAINER -- /bin/bash
# Or if bash not available
kubectl exec -it $POD_NAME -n $NAMESPACE -c $CONTAINER -- /bin/sh

# Run single command
kubectl exec $POD_NAME -n $NAMESPACE -c $CONTAINER -- ls -la /app

# Check environment variables
kubectl exec $POD_NAME -n $NAMESPACE -c $CONTAINER -- env

# Check processes
kubectl exec $POD_NAME -n $NAMESPACE -c $CONTAINER -- ps aux
```
Executes commands inside the container.

### 4. Debug with ephemeral container (K8s 1.23+)
```bash
# Add debug container with tools
kubectl debug $POD_NAME -n $NAMESPACE -it --image=busybox --share-processes

# Debug with specific container
kubectl debug $POD_NAME -n $NAMESPACE -it --image=nicolaka/netshoot --target=$CONTAINER

# Copy pod with debug image
kubectl debug $POD_NAME -n $NAMESPACE --copy-to=debug-$POD_NAME --container=debug --image=ubuntu
```
Uses ephemeral containers for debugging.

### 5. Check resource usage
```bash
# CPU and memory usage
kubectl top pod $POD_NAME -n $NAMESPACE --containers

# Resource limits and requests
kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.spec.containers[*].resources}'

# Detailed resource info
kubectl get pod $POD_NAME -n $NAMESPACE -o yaml | grep -A 10 resources:
```
Monitors resource consumption.

### 6. Network debugging
```bash
# Test DNS resolution from pod
kubectl exec $POD_NAME -n $NAMESPACE -- nslookup kubernetes.default

# Check network connectivity
kubectl exec $POD_NAME -n $NAMESPACE -- wget -O- http://service-name

# List network interfaces
kubectl exec $POD_NAME -n $NAMESPACE -- ip addr

# Check port connectivity
kubectl exec $POD_NAME -n $NAMESPACE -- nc -zv service-name 80
```
Debugs networking issues.

### 7. File system inspection
```bash
# Check disk usage
kubectl exec $POD_NAME -n $NAMESPACE -- df -h

# List files in specific directory
kubectl exec $POD_NAME -n $NAMESPACE -- ls -la /var/log

# Check file permissions
kubectl exec $POD_NAME -n $NAMESPACE -- stat /app/config.yaml

# View file content
kubectl exec $POD_NAME -n $NAMESPACE -- cat /etc/hosts
```
Inspects file system state.

### 8. Common debugging patterns
```bash
# Check if config is mounted correctly
kubectl exec $POD_NAME -n $NAMESPACE -- ls -la /etc/config

# Verify secrets are accessible
kubectl exec $POD_NAME -n $NAMESPACE -- ls -la /etc/secrets

# Test database connectivity
kubectl exec $POD_NAME -n $NAMESPACE -- nc -zv postgres-service 5432

# Check health endpoint
kubectl exec $POD_NAME -n $NAMESPACE -- curl -s localhost:8080/health
```
Common debugging scenarios.

## Validation
- Pod events show clear error messages
- Logs indicate the root cause
- Commands execute successfully in pod
- Resource usage is within limits
- Network connectivity works

## Error Handling
- **"container not found"** - List containers: `kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.spec.containers[*].name}'`
- **"pod is pending"** - Check events and node resources
- **"exec failed"** - Pod might be crashing, check with `--previous` logs
- **"permission denied"** - Try different shell or check security context

## Safety Notes
- Be careful with exec commands in production
- Some commands might impact pod performance
- Avoid modifying files unless necessary
- Consider read-only commands first
- Use debug containers instead of modifying pod

## Examples
- **Debug crashing pod**
  ```bash
  kubectl logs frontend-5d8c7b9f-x2n4k --previous
  kubectl describe pod frontend-5d8c7b9f-x2n4k
  ```

- **Interactive debugging**
  ```bash
  kubectl exec -it backend-7c9f8b6d5-m3n2p -- /bin/bash
  ```

- **Network troubleshooting**
  ```bash
  kubectl debug api-pod --image=nicolaka/netshoot --share-processes
  ```

- **Check multi-container pod**
  ```bash
  kubectl logs pod-name -c sidecar-container -f
  ```