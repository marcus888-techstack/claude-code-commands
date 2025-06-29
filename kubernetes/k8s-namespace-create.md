# K8s Namespace Create

## Purpose
Create and configure a Kubernetes namespace for organizing and isolating resources

## Context
Use namespaces to separate environments (dev, staging, prod), teams, or projects. Namespaces provide scope for names, resource quotas, and access control boundaries.

## Parameters
- `$NAMESPACE` - Name of the namespace to create
  - Required
  - Example: `development`, `team-frontend`, `project-x`
- `$LABELS` - Labels to apply to namespace
  - Optional
  - Example: `env=dev,team=frontend`
- `$RESOURCE_QUOTA` - Whether to set resource limits
  - Optional
  - Default: `false`
  - Options: `true`, `false`

## Steps

### 1. Create basic namespace
```bash
# Create namespace with kubectl
kubectl create namespace $NAMESPACE

# Alternative: Create from YAML
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: $NAMESPACE
  labels:
    name: $NAMESPACE
EOF

# Create with labels
kubectl create namespace $NAMESPACE \
  --labels=env=development,team=backend
```
Creates the namespace.

### 2. Configure namespace with annotations
```bash
# Add namespace metadata
kubectl annotate namespace $NAMESPACE \
  description="Namespace for development environment" \
  owner="platform-team" \
  created-by="$(whoami)" \
  created-date="$(date +%Y-%m-%d)"

# Add labels for organization
kubectl label namespace $NAMESPACE \
  environment=development \
  team=frontend \
  cost-center=engineering
```
Adds metadata for management.

### 3. Set resource quotas
```bash
# Create ResourceQuota
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: $NAMESPACE-quota
  namespace: $NAMESPACE
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "10"
    services: "10"
    services.loadbalancers: "2"
    pods: "50"
EOF

# Verify quota
kubectl describe resourcequota -n $NAMESPACE
```
Limits resource consumption.

### 4. Configure default limits
```bash
# Set default container limits
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: $NAMESPACE-limits
  namespace: $NAMESPACE
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
  - max:
      storage: 10Gi
    type: PersistentVolumeClaim
EOF

# Check limits
kubectl describe limitrange -n $NAMESPACE
```
Sets default resource constraints.

### 5. Configure RBAC
```bash
# Create namespace admin role
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: namespace-admin
  namespace: $NAMESPACE
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: namespace-admin-binding
  namespace: $NAMESPACE
subjects:
- kind: User
  name: $USER_EMAIL
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: namespace-admin
  apiGroup: rbac.authorization.k8s.io
EOF
```
Sets up access control.

### 6. Create network policies
```bash
# Default deny all ingress
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: $NAMESPACE
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF

# Allow from same namespace
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
  namespace: $NAMESPACE
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
EOF
```
Configures network isolation.

### 7. Set up namespace defaults
```bash
# Create default ServiceAccount
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
  namespace: $NAMESPACE
automountServiceAccountToken: false
EOF

# Create ConfigMap for defaults
kubectl create configmap namespace-defaults \
  --from-literal=environment=$NAMESPACE \
  --from-literal=region=us-east-1 \
  --from-literal=log-level=info \
  -n $NAMESPACE
```
Sets namespace-wide defaults.

### 8. Verify and use namespace
```bash
# List all namespaces
kubectl get namespaces

# Get namespace details
kubectl describe namespace $NAMESPACE

# Set as default namespace
kubectl config set-context --current --namespace=$NAMESPACE

# Quick switch between namespaces
alias kns='kubectl config set-context --current --namespace'
kns $NAMESPACE

# List resources in namespace
kubectl get all -n $NAMESPACE
```
Verifies creation and switches context.

## Validation
- Namespace exists and is Active
- Labels and annotations are set
- ResourceQuota is applied (if configured)
- LimitRange is active
- RBAC policies work correctly

## Error Handling
- **"namespace already exists"** - Use `kubectl get ns $NAMESPACE` to check
- **"forbidden"** - Check RBAC permissions
- **"quota exceeded"** - Review ResourceQuota limits
- **"invalid name"** - Use lowercase, alphanumeric, and hyphens only

## Safety Notes
- Plan namespace strategy before creating
- Use consistent naming conventions
- Set resource quotas to prevent overuse
- Configure RBAC from the start
- Document namespace purpose and owner

## Examples
- **Development namespace**
  ```bash
  kubectl create namespace dev-frontend --labels=env=dev,team=frontend
  ```

- **Production with quotas**
  ```bash
  kubectl create namespace production
  kubectl apply -f production-quota.yaml
  ```

- **Team namespace with RBAC**
  ```bash
  kubectl create namespace team-data
  kubectl apply -f team-data-rbac.yaml
  ```

- **Temporary namespace**
  ```bash
  kubectl create namespace test-feature-123 --labels=temporary=true,expires=$(date -d '+7 days' +%Y-%m-%d)
  ```