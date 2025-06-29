# K8s Deployment Rollout

## Purpose
Perform a rolling update of a Kubernetes deployment with zero downtime, including rollback capabilities

## Context
Use when deploying new versions of applications, updating configurations, or rolling back problematic releases. This ensures continuous availability during updates by gradually replacing old pods with new ones.

## Parameters
- `$DEPLOYMENT_NAME` - Name of the deployment
  - Required
  - Example: `frontend`, `api-server`
- `$IMAGE` - New container image with tag
  - Required for updates
  - Example: `myapp:v2.0`, `gcr.io/project/app:latest`
- `$NAMESPACE` - Kubernetes namespace
  - Optional
  - Default: `default`
  - Example: `production`, `staging`

## Steps

### 1. Check current deployment status
```bash
# Get deployment details
kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE

# View current image
kubectl describe deployment $DEPLOYMENT_NAME -n $NAMESPACE | grep Image:

# Check rollout history
kubectl rollout history deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Get current replica count
kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.spec.replicas}'
```
Verifies deployment state before update.

### 2. Update deployment image
```bash
# Update image for deployment
kubectl set image deployment/$DEPLOYMENT_NAME \
  $CONTAINER_NAME=$IMAGE \
  -n $NAMESPACE \
  --record

# Alternative: Edit deployment directly
kubectl edit deployment $DEPLOYMENT_NAME -n $NAMESPACE

# Using patch for specific updates
kubectl patch deployment $DEPLOYMENT_NAME -n $NAMESPACE \
  -p '{"spec":{"template":{"spec":{"containers":[{"name":"app","image":"'$IMAGE'"}]}}}}'
```
Triggers the rolling update.

### 3. Monitor rollout progress
```bash
# Watch rollout status
kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Monitor pods during rollout
kubectl get pods -n $NAMESPACE -l app=$DEPLOYMENT_NAME -w

# Check deployment conditions
kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE \
  -o jsonpath='{.status.conditions[?(@.type=="Progressing")]}'

# View detailed rollout events
kubectl describe deployment $DEPLOYMENT_NAME -n $NAMESPACE | tail -20
```
Tracks update progress in real-time.

### 4. Verify new version
```bash
# Check all pods are running new version
kubectl get pods -n $NAMESPACE -l app=$DEPLOYMENT_NAME \
  -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

# Test application endpoint
kubectl run test-curl --rm -it --image=curlimages/curl -- \
  curl http://$SERVICE_NAME.$NAMESPACE/health

# Check pod logs for errors
kubectl logs -n $NAMESPACE -l app=$DEPLOYMENT_NAME --tail=50
```
Confirms successful deployment.

### 5. Rollback if needed
```bash
# Immediate rollback to previous version
kubectl rollout undo deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Rollback to specific revision
kubectl rollout undo deployment/$DEPLOYMENT_NAME -n $NAMESPACE --to-revision=2

# Check rollback status
kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Pause rollout if issues detected
kubectl rollout pause deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Resume after fixing
kubectl rollout resume deployment/$DEPLOYMENT_NAME -n $NAMESPACE
```
Reverts to previous version if problems occur.

### 6. Advanced rollout strategies
```bash
# Set deployment strategy
kubectl patch deployment $DEPLOYMENT_NAME -n $NAMESPACE -p \
  '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":"25%","maxUnavailable":"25%"}}}}'

# Scale deployment during rollout
kubectl scale deployment $DEPLOYMENT_NAME -n $NAMESPACE --replicas=10

# Update multiple deployments
kubectl set image deployment/frontend frontend=$IMAGE \
  deployment/backend backend=$IMAGE \
  -n $NAMESPACE
```
Configures advanced deployment options.

### 7. Canary deployment approach
```bash
# Create canary deployment
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $DEPLOYMENT_NAME-canary
  namespace: $NAMESPACE
spec:
  replicas: 1
  selector:
    matchLabels:
      app: $DEPLOYMENT_NAME
      version: canary
  template:
    metadata:
      labels:
        app: $DEPLOYMENT_NAME
        version: canary
    spec:
      containers:
      - name: app
        image: $IMAGE
EOF

# Monitor canary
kubectl logs -n $NAMESPACE -l version=canary -f

# Gradually increase canary traffic
kubectl scale deployment $DEPLOYMENT_NAME-canary -n $NAMESPACE --replicas=3
```
Implements canary deployment pattern.

### 8. Post-deployment validation
```bash
# Run smoke tests
kubectl run smoke-test --rm -it --image=busybox -- \
  wget -O- http://$SERVICE_NAME.$NAMESPACE/api/health

# Check metrics (if metrics-server installed)
kubectl top pods -n $NAMESPACE -l app=$DEPLOYMENT_NAME

# Verify horizontal pod autoscaler
kubectl get hpa -n $NAMESPACE

# Check deployment annotations
kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE \
  -o jsonpath='{.metadata.annotations}'
```
Validates deployment health.

## Validation
- All pods are running with new image
- No error events in deployment
- Application endpoints respond correctly
- Rollout completed successfully
- Previous versions available for rollback

## Error Handling
- **"ImagePullBackOff"** - Check image name and registry credentials
- **"Insufficient resources"** - Scale down or add nodes
- **"Readiness probe failed"** - Review health check configuration
- **"Rollout stuck"** - Check pod events and logs

## Safety Notes
- Always record deployments with `--record`
- Test in staging environment first
- Have rollback plan ready
- Monitor application metrics during rollout
- Use readiness probes to prevent bad deploys

## Examples
- **Simple image update**
  ```bash
  kubectl set image deployment/frontend app=frontend:v2.0 --record
  kubectl rollout status deployment/frontend
  ```

- **Rollback deployment**
  ```bash
  kubectl rollout undo deployment/api-server -n production
  ```

- **Canary deployment**
  ```bash
  kubectl set image deployment/backend-canary app=backend:v3.0
  kubectl scale deployment backend-canary --replicas=2
  ```

- **Update with strategy**
  ```bash
  kubectl patch deployment web -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":"50%","maxUnavailable":"0"}}}}'
  ```