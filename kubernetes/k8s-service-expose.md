# K8s Service Expose

## Purpose
Expose a deployment, pod, or replicaset as a Kubernetes service to enable network access

## Context
Use when you need to make your application accessible within the cluster or externally. Services provide stable networking endpoints, load balancing, and service discovery for your pods.

## Parameters
- `$RESOURCE_NAME` - Name of deployment/pod to expose
  - Required
  - Example: `frontend`, `api-deployment`
- `$SERVICE_TYPE` - Type of service to create
  - Optional
  - Default: `ClusterIP`
  - Options: `ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`
- `$PORT` - Service port
  - Required
  - Example: `80`, `8080`, `3000`
- `$TARGET_PORT` - Container port
  - Optional
  - Default: Same as `$PORT`
  - Example: `8080`, `3000`

## Steps

### 1. Expose deployment as service
```bash
# Basic service exposure
kubectl expose deployment $RESOURCE_NAME \
  --name=$SERVICE_NAME \
  --port=$PORT \
  --target-port=$TARGET_PORT \
  --type=$SERVICE_TYPE \
  -n $NAMESPACE

# Expose with specific selector
kubectl expose deployment $RESOURCE_NAME \
  --port=$PORT \
  --target-port=$TARGET_PORT \
  --selector="app=myapp,tier=frontend" \
  -n $NAMESPACE

# Expose with session affinity
kubectl expose deployment $RESOURCE_NAME \
  --port=$PORT \
  --session-affinity=ClientIP \
  -n $NAMESPACE
```
Creates service for deployment.

### 2. Create service with YAML
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: $SERVICE_NAME
  namespace: $NAMESPACE
  labels:
    app: $RESOURCE_NAME
spec:
  type: $SERVICE_TYPE
  selector:
    app: $RESOURCE_NAME
  ports:
    - port: $PORT
      targetPort: $TARGET_PORT
      protocol: TCP
      name: http
  sessionAffinity: None
EOF
```
Creates service using manifest.

### 3. Expose with NodePort
```bash
# Expose with specific NodePort
kubectl expose deployment $RESOURCE_NAME \
  --type=NodePort \
  --port=$PORT \
  --target-port=$TARGET_PORT \
  --node-port=30080 \
  -n $NAMESPACE

# Get assigned NodePort
kubectl get service $SERVICE_NAME -n $NAMESPACE \
  -o jsonpath='{.spec.ports[0].nodePort}'

# Access via NodePort
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
curl http://$NODE_IP:30080
```
Exposes service on all nodes.

### 4. Create LoadBalancer service
```bash
# Expose as LoadBalancer
kubectl expose deployment $RESOURCE_NAME \
  --type=LoadBalancer \
  --port=$PORT \
  --target-port=$TARGET_PORT \
  -n $NAMESPACE

# Wait for external IP
kubectl get service $SERVICE_NAME -n $NAMESPACE -w

# Get LoadBalancer IP/hostname
kubectl get service $SERVICE_NAME -n $NAMESPACE \
  -o jsonpath='{.status.loadBalancer.ingress[0].*}'
```
Creates cloud load balancer.

### 5. Configure service endpoints
```bash
# Create headless service
kubectl expose deployment $RESOURCE_NAME \
  --cluster-ip=None \
  --port=$PORT \
  -n $NAMESPACE

# Multi-port service
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: $SERVICE_NAME
  namespace: $NAMESPACE
spec:
  selector:
    app: $RESOURCE_NAME
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: https
      port: 443
      targetPort: 8443
    - name: metrics
      port: 9090
      targetPort: 9090
EOF
```
Advanced service configurations.

### 6. Test service connectivity
```bash
# Test from within cluster
kubectl run test-pod --rm -it --image=busybox -- \
  wget -O- http://$SERVICE_NAME.$NAMESPACE:$PORT

# DNS resolution test
kubectl run test-dns --rm -it --image=busybox -- \
  nslookup $SERVICE_NAME.$NAMESPACE

# Port forwarding for local testing
kubectl port-forward service/$SERVICE_NAME 8080:$PORT -n $NAMESPACE

# Check service endpoints
kubectl get endpoints $SERVICE_NAME -n $NAMESPACE
```
Verifies service is working.

### 7. Configure Ingress (if needed)
```bash
# Create Ingress for external access
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: $RESOURCE_NAME-ingress
  namespace: $NAMESPACE
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: $HOST_NAME
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: $SERVICE_NAME
            port:
              number: $PORT
EOF

# Check Ingress status
kubectl get ingress -n $NAMESPACE
```
Configures HTTP(S) routing.

### 8. Service monitoring and troubleshooting
```bash
# Check service details
kubectl describe service $SERVICE_NAME -n $NAMESPACE

# View service YAML
kubectl get service $SERVICE_NAME -n $NAMESPACE -o yaml

# Check if pods match selector
kubectl get pods -n $NAMESPACE -l app=$RESOURCE_NAME

# Monitor service endpoints
kubectl get endpoints $SERVICE_NAME -n $NAMESPACE -w

# Check iptables rules (advanced)
kubectl run tmp-shell --rm -it --image=nicolaka/netshoot -- \
  iptables -t nat -L KUBE-SERVICES
```
Debugs service issues.

## Validation
- Service created successfully
- Endpoints show pod IPs
- Service accessible from pods
- External access works (if applicable)
- DNS resolution functions

## Error Handling
- **"No endpoints"** - Check selector matches pod labels
- **"Connection refused"** - Verify target port is correct
- **"External IP pending"** - LoadBalancer provisioning in progress
- **"Service not found"** - Check namespace and service name

## Safety Notes
- Use ClusterIP for internal services
- Restrict NodePort range for security
- LoadBalancer incurs cloud costs
- Always specify target port explicitly
- Use NetworkPolicies to restrict access

## Examples
- **Basic ClusterIP service**
  ```bash
  kubectl expose deployment webapp --port=80 --target-port=8080
  ```

- **NodePort for testing**
  ```bash
  kubectl expose deployment frontend --type=NodePort --port=80 --node-port=30080
  ```

- **LoadBalancer for production**
  ```bash
  kubectl expose deployment api --type=LoadBalancer --port=443 --target-port=8443
  ```

- **Headless service for StatefulSet**
  ```bash
  kubectl expose statefulset mongodb --cluster-ip=None --port=27017
  ```