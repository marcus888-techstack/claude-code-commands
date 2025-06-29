# K8s Helm Deploy

## Purpose
Deploy applications to Kubernetes using Helm charts for templating and package management

## Context
Use Helm to manage complex Kubernetes applications with templated manifests, versioning, and easy rollbacks. Helm charts package all resources needed for an application and allow configuration through values files.

## Parameters
- `$RELEASE_NAME` - Name for the Helm release
  - Required
  - Example: `my-app`, `backend-api`, `frontend`
- `$CHART` - Helm chart to install
  - Required
  - Example: `stable/nginx`, `./my-chart`, `bitnami/postgresql`
- `$NAMESPACE` - Kubernetes namespace
  - Optional
  - Default: `default`

## Steps

### 1. Add Helm repositories
```bash
# Add official Helm stable charts
helm repo add stable https://charts.helm.sh/stable

# Add Bitnami repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Add ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Update repositories
helm repo update

# List repositories
helm repo list

# Search for charts
helm search repo nginx
helm search repo postgres --versions
```
Configures Helm repositories.

### 2. Install Helm chart
```bash
# Basic install
helm install $RELEASE_NAME $CHART -n $NAMESPACE

# Install with custom values
helm install $RELEASE_NAME $CHART \
  --values values.yaml \
  -n $NAMESPACE

# Install with inline values
helm install $RELEASE_NAME $CHART \
  --set service.type=LoadBalancer \
  --set persistence.enabled=true \
  --set persistence.size=10Gi \
  -n $NAMESPACE

# Install specific version
helm install $RELEASE_NAME $CHART \
  --version 1.2.3 \
  -n $NAMESPACE

# Install with wait
helm install $RELEASE_NAME $CHART \
  --wait --timeout 5m \
  -n $NAMESPACE
```
Installs Helm charts.

### 3. Create values files
```bash
# Development values
cat <<EOF > values-dev.yaml
replicaCount: 1
image:
  repository: myapp
  tag: dev
  pullPolicy: Always
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: true
  host: dev.example.com
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
EOF

# Production values
cat <<EOF > values-prod.yaml
replicaCount: 3
image:
  repository: myapp
  tag: v1.0.0
  pullPolicy: IfNotPresent
service:
  type: LoadBalancer
  port: 80
ingress:
  enabled: true
  host: api.example.com
  tls:
    enabled: true
resources:
  limits:
    cpu: 2
    memory: 2Gi
  requests:
    cpu: 1
    memory: 1Gi
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
EOF
```
Creates environment-specific values.

### 4. Upgrade releases
```bash
# Upgrade with new values
helm upgrade $RELEASE_NAME $CHART \
  --values values-prod.yaml \
  -n $NAMESPACE

# Upgrade with new image
helm upgrade $RELEASE_NAME $CHART \
  --set image.tag=v2.0.0 \
  --reuse-values \
  -n $NAMESPACE

# Upgrade or install
helm upgrade --install $RELEASE_NAME $CHART \
  --values values.yaml \
  -n $NAMESPACE

# Force upgrade
helm upgrade $RELEASE_NAME $CHART \
  --force --recreate-pods \
  -n $NAMESPACE

# Dry run
helm upgrade $RELEASE_NAME $CHART \
  --dry-run --debug \
  -n $NAMESPACE
```
Updates existing releases.

### 5. Manage releases
```bash
# List releases
helm list -n $NAMESPACE
helm list --all-namespaces

# Get release info
helm status $RELEASE_NAME -n $NAMESPACE
helm get values $RELEASE_NAME -n $NAMESPACE
helm get manifest $RELEASE_NAME -n $NAMESPACE

# Release history
helm history $RELEASE_NAME -n $NAMESPACE

# Rollback
helm rollback $RELEASE_NAME 1 -n $NAMESPACE

# Test release
helm test $RELEASE_NAME -n $NAMESPACE
```
Manages Helm releases.

### 6. Create custom chart
```bash
# Create new chart
helm create $CHART_NAME

# Chart structure
tree $CHART_NAME/
# ├── Chart.yaml
# ├── charts/
# ├── templates/
# │   ├── NOTES.txt
# │   ├── _helpers.tpl
# │   ├── deployment.yaml
# │   ├── ingress.yaml
# │   ├── service.yaml
# │   └── tests/
# └── values.yaml

# Modify Chart.yaml
cat <<EOF > $CHART_NAME/Chart.yaml
apiVersion: v2
name: $CHART_NAME
description: A Helm chart for my application
type: application
version: 0.1.0
appVersion: "1.0"
dependencies:
  - name: postgresql
    version: "11.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
EOF

# Package chart
helm package $CHART_NAME

# Install local chart
helm install $RELEASE_NAME ./$CHART_NAME
```
Creates custom Helm charts.

### 7. Advanced deployment patterns
```bash
# Blue-green deployment
helm install $RELEASE_NAME-blue $CHART \
  --set version=blue \
  --values values-blue.yaml \
  -n $NAMESPACE

helm install $RELEASE_NAME-green $CHART \
  --set version=green \
  --values values-green.yaml \
  -n $NAMESPACE

# Canary deployment
helm upgrade $RELEASE_NAME $CHART \
  --set canary.enabled=true \
  --set canary.weight=20 \
  -n $NAMESPACE

# Multi-environment
for env in dev staging prod; do
  helm upgrade --install app-$env $CHART \
    --values values-$env.yaml \
    -n $env
done

# With secrets
kubectl create secret generic app-secrets \
  --from-literal=api-key=$API_KEY \
  -n $NAMESPACE

helm install $RELEASE_NAME $CHART \
  --set existingSecret=app-secrets \
  -n $NAMESPACE
```
Advanced deployment strategies.

### 8. Troubleshooting
```bash
# Debug installation
helm install $RELEASE_NAME $CHART \
  --debug --dry-run \
  -n $NAMESPACE

# Get detailed errors
helm status $RELEASE_NAME -n $NAMESPACE
kubectl describe pods -l app.kubernetes.io/instance=$RELEASE_NAME -n $NAMESPACE

# Check rendered templates
helm template $RELEASE_NAME $CHART \
  --values values.yaml > rendered.yaml

# Lint chart
helm lint $CHART

# Delete stuck release
helm delete $RELEASE_NAME -n $NAMESPACE
helm delete $RELEASE_NAME --purge  # Helm 2

# Fix failed release
helm rollback $RELEASE_NAME 0 -n $NAMESPACE  # Rollback to previous
helm delete $RELEASE_NAME -n $NAMESPACE
helm install $RELEASE_NAME $CHART -n $NAMESPACE
```
Troubleshoots Helm issues.

## Validation
- Release shows as DEPLOYED
- All pods are running
- Services are accessible
- Ingress routes work
- Resource limits applied

## Error Handling
- **"release not found"** - Check release name and namespace
- **"cannot re-use a name"** - Delete existing release first
- **"no repository"** - Add repository with `helm repo add`
- **"validation failed"** - Check values against chart schema

## Safety Notes
- Always use `--dry-run` first in production
- Keep values files in version control
- Use `--atomic` flag for automatic rollback
- Test charts in staging first
- Document custom values

## Examples
- **Install NGINX**
  ```bash
  helm install my-nginx bitnami/nginx \
    --set service.type=LoadBalancer \
    -n web
  ```

- **Deploy with custom values**
  ```bash
  helm install backend ./my-app \
    --values values-prod.yaml \
    --set image.tag=$VERSION \
    -n production
  ```

- **Upgrade application**
  ```bash
  helm upgrade frontend frontend-chart \
    --reuse-values \
    --set image.tag=v2.0.0 \
    --wait
  ```

- **Multi-environment deploy**
  ```bash
  helm upgrade --install app-dev ./chart \
    --values envs/dev.yaml \
    -n development
  ```