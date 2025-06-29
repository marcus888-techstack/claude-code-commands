# K8s Scale Replicas

## Purpose
Scale the number of replicas for deployments, statefulsets, or replicasets to handle load changes

## Context
Use to manually adjust application capacity, handle traffic spikes, reduce costs during low usage, or prepare for maintenance. Can be combined with Horizontal Pod Autoscaler (HPA) for automatic scaling.

## Parameters
- `$RESOURCE_NAME` - Name of the resource to scale
  - Required
  - Example: `frontend`, `api-server`, `worker`
- `$REPLICAS` - Target number of replicas
  - Required
  - Example: `3`, `10`, `0` (to stop)
- `$RESOURCE_TYPE` - Type of resource
  - Optional
  - Default: `deployment`
  - Options: `deployment`, `statefulset`, `replicaset`

## Steps

### 1. Scale deployment
```bash
# Scale to specific number
kubectl scale deployment $RESOURCE_NAME --replicas=$REPLICAS -n $NAMESPACE

# Scale multiple deployments
kubectl scale deployment frontend backend worker --replicas=3 -n $NAMESPACE

# Scale with selector
kubectl scale deployment -l app=myapp --replicas=5 -n $NAMESPACE

# Scale to zero (stop all pods)
kubectl scale deployment $RESOURCE_NAME --replicas=0 -n $NAMESPACE
```
Basic scaling operations.

### 2. Monitor scaling progress
```bash
# Watch replica changes
kubectl get deployment $RESOURCE_NAME -n $NAMESPACE -w

# Check rollout status
kubectl rollout status deployment/$RESOURCE_NAME -n $NAMESPACE

# View pods during scaling
kubectl get pods -l app=$RESOURCE_NAME -n $NAMESPACE -w

# Check events
kubectl get events --field-selector involvedObject.name=$RESOURCE_NAME -n $NAMESPACE
```
Monitors scaling activity.

### 3. Scale with conditions
```bash
# Scale only if current replicas match
kubectl scale deployment $RESOURCE_NAME \
  --current-replicas=2 \
  --replicas=5 \
  -n $NAMESPACE

# Scale based on resource availability
if kubectl top nodes | awk 'NR>1 {sum+=$3} END {print (sum/NR<70)}'; then
  kubectl scale deployment $RESOURCE_NAME --replicas=$((REPLICAS+2)) -n $NAMESPACE
fi

# Gradual scaling
for i in $(seq 1 $REPLICAS); do
  kubectl scale deployment $RESOURCE_NAME --replicas=$i -n $NAMESPACE
  sleep 10
done
```
Conditional scaling strategies.

### 4. Configure Horizontal Pod Autoscaler
```bash
# Create HPA for CPU
kubectl autoscale deployment $RESOURCE_NAME \
  --min=2 \
  --max=10 \
  --cpu-percent=80 \
  -n $NAMESPACE

# Create HPA with custom metrics
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: $RESOURCE_NAME-hpa
  namespace: $NAMESPACE
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: $RESOURCE_NAME
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
EOF

# Check HPA status
kubectl get hpa -n $NAMESPACE
```
Sets up automatic scaling.

### 5. Scale StatefulSets
```bash
# Scale statefulset (ordered)
kubectl scale statefulset $RESOURCE_NAME --replicas=$REPLICAS -n $NAMESPACE

# Patch for parallel scaling
kubectl patch statefulset $RESOURCE_NAME -n $NAMESPACE \
  -p '{"spec":{"podManagementPolicy":"Parallel"}}'

# Scale down carefully
kubectl scale statefulset $RESOURCE_NAME --replicas=$((REPLICAS-1)) -n $NAMESPACE

# Wait for each pod
kubectl wait --for=condition=ready pod/${RESOURCE_NAME}-$((REPLICAS-1)) -n $NAMESPACE
```
Handles stateful scaling.

### 6. Implement scaling policies
```bash
# Vertical Pod Autoscaler
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: $RESOURCE_NAME-vpa
  namespace: $NAMESPACE
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: $RESOURCE_NAME
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: '*'
      maxAllowed:
        cpu: 2
        memory: 4Gi
EOF

# Pod Disruption Budget for safe scaling
cat <<EOF | kubectl apply -f -
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: $RESOURCE_NAME-pdb
  namespace: $NAMESPACE
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: $RESOURCE_NAME
EOF
```
Advanced scaling policies.

### 7. Resource-aware scaling
```bash
# Check node resources before scaling
NODE_CAPACITY=$(kubectl get nodes -o json | \
  jq '[.items[].status.allocatable.cpu | rtrimstr("m") | tonumber] | add')

POD_CPU=$(kubectl get deployment $RESOURCE_NAME -n $NAMESPACE \
  -o jsonpath='{.spec.template.spec.containers[0].resources.requests.cpu}' | \
  sed 's/m//')

MAX_REPLICAS=$((NODE_CAPACITY / POD_CPU))
echo "Maximum replicas based on CPU: $MAX_REPLICAS"

# Scale with resource check
if [ $REPLICAS -le $MAX_REPLICAS ]; then
  kubectl scale deployment $RESOURCE_NAME --replicas=$REPLICAS -n $NAMESPACE
else
  echo "Warning: Not enough resources for $REPLICAS replicas"
fi
```
Resource-aware scaling.

### 8. Scaling automation scripts
```bash
# Scale based on time of day
HOUR=$(date +%H)
if [ $HOUR -ge 9 ] && [ $HOUR -le 17 ]; then
  # Business hours
  kubectl scale deployment $RESOURCE_NAME --replicas=10 -n $NAMESPACE
else
  # Off hours
  kubectl scale deployment $RESOURCE_NAME --replicas=3 -n $NAMESPACE
fi

# Scale based on queue length (example)
QUEUE_LENGTH=$(kubectl exec -n $NAMESPACE deployment/$RESOURCE_NAME -- \
  sh -c 'redis-cli llen job_queue')

if [ $QUEUE_LENGTH -gt 1000 ]; then
  kubectl scale deployment worker --replicas=20 -n $NAMESPACE
elif [ $QUEUE_LENGTH -lt 100 ]; then
  kubectl scale deployment worker --replicas=5 -n $NAMESPACE
fi

# Rolling scale update
scale_gradually() {
  local current=$(kubectl get deployment $1 -n $2 -o jsonpath='{.spec.replicas}')
  local target=$3
  local step=$([ $current -lt $target ] && echo 1 || echo -1)
  
  while [ $current -ne $target ]; do
    current=$((current + step))
    kubectl scale deployment $1 --replicas=$current -n $2
    sleep 30
  done
}

scale_gradually $RESOURCE_NAME $NAMESPACE $REPLICAS
```
Automated scaling scripts.

## Validation
- Replica count matches target
- All pods are running and ready
- No pending pods due to resources
- Load is distributed evenly
- Autoscaler is working (if configured)

## Error Handling
- **"insufficient resources"** - Check node capacity
- **"scale would violate PDB"** - Adjust PodDisruptionBudget
- **"timeout waiting"** - Check pod startup issues
- **"forbidden"** - Verify RBAC permissions

## Safety Notes
- Don't scale to zero in production without confirmation
- Check PodDisruptionBudget before scaling down
- Monitor resource usage during scaling
- Consider gradual scaling for large changes
- Test autoscaling policies in staging first

## Examples
- **Quick scale up**
  ```bash
  kubectl scale deployment frontend --replicas=10 -n production
  ```

- **Scale to zero**
  ```bash
  kubectl scale deployment batch-job --replicas=0 -n jobs
  ```

- **Autoscale setup**
  ```bash
  kubectl autoscale deployment api --min=3 --max=20 --cpu-percent=70
  ```

- **Conditional scaling**
  ```bash
  kubectl scale deployment worker --current-replicas=5 --replicas=10
  ```