# K8s ConfigMap Create

## Purpose
Create and manage ConfigMaps to store non-sensitive configuration data for applications

## Context
Use ConfigMaps to decouple configuration from container images, making applications portable across environments. ConfigMaps can store key-value pairs, configuration files, or command-line arguments.

## Parameters
- `$CONFIGMAP_NAME` - Name of the ConfigMap
  - Required
  - Example: `app-config`, `nginx-conf`, `env-vars`
- `$SOURCE` - Source of configuration data
  - Required
  - Options: `literal`, `file`, `directory`, `env-file`
- `$NAMESPACE` - Target namespace
  - Optional
  - Default: `default`

## Steps

### 1. Create from literal values
```bash
# Single key-value pair
kubectl create configmap $CONFIGMAP_NAME \
  --from-literal=key1=value1 \
  -n $NAMESPACE

# Multiple key-value pairs
kubectl create configmap $CONFIGMAP_NAME \
  --from-literal=database_host=postgres \
  --from-literal=database_port=5432 \
  --from-literal=log_level=info \
  --from-literal=max_connections=100 \
  -n $NAMESPACE

# With special characters
kubectl create configmap $CONFIGMAP_NAME \
  --from-literal=connection_string='postgres://user:pass@host:5432/db' \
  --from-literal=api_key='sk-abc123xyz789' \
  -n $NAMESPACE
```
Creates ConfigMap from literals.

### 2. Create from files
```bash
# From single file
kubectl create configmap $CONFIGMAP_NAME \
  --from-file=config.properties \
  -n $NAMESPACE

# From file with custom key
kubectl create configmap $CONFIGMAP_NAME \
  --from-file=app.conf=config/application.conf \
  -n $NAMESPACE

# From multiple files
kubectl create configmap $CONFIGMAP_NAME \
  --from-file=server.xml \
  --from-file=logging.properties \
  --from-file=database.properties \
  -n $NAMESPACE

# From directory
kubectl create configmap $CONFIGMAP_NAME \
  --from-file=config/ \
  -n $NAMESPACE
```
Creates ConfigMap from files.

### 3. Create from env file
```bash
# Create .env file
cat <<EOF > app.env
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=myapp
LOG_LEVEL=debug
CACHE_SIZE=1000
EOF

# Create ConfigMap from env file
kubectl create configmap $CONFIGMAP_NAME \
  --from-env-file=app.env \
  -n $NAMESPACE

# From multiple env files
kubectl create configmap $CONFIGMAP_NAME \
  --from-env-file=app.env \
  --from-env-file=secrets.env \
  -n $NAMESPACE
```
Creates from environment files.

### 4. Create with YAML manifest
```bash
# Basic ConfigMap
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: $CONFIGMAP_NAME
  namespace: $NAMESPACE
  labels:
    app: myapp
    environment: production
data:
  # Simple key-value pairs
  database_host: "postgres.example.com"
  database_port: "5432"
  
  # Multi-line values
  application.properties: |
    server.port=8080
    server.context-path=/api
    logging.level.root=INFO
    
  # JSON configuration
  config.json: |
    {
      "apiUrl": "https://api.example.com",
      "timeout": 30,
      "retries": 3
    }
EOF
```
Creates using YAML manifests.

### 5. Use ConfigMap in pods
```bash
# As environment variables
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: $NAMESPACE
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    # Single environment variable
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: $CONFIGMAP_NAME
          key: database_host
    # All ConfigMap data as env vars
    envFrom:
    - configMapRef:
        name: $CONFIGMAP_NAME
        prefix: APP_  # Optional prefix
EOF

# As volume mount
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-volume
  namespace: $NAMESPACE
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: config
    configMap:
      name: $CONFIGMAP_NAME
      # Optional: specific items
      items:
      - key: application.properties
        path: app.properties
      - key: config.json
        path: app-config.json
EOF
```
Mounts ConfigMap in pods.

### 6. Update ConfigMap
```bash
# Replace entire ConfigMap
kubectl create configmap $CONFIGMAP_NAME \
  --from-file=new-config/ \
  --dry-run=client -o yaml | \
  kubectl replace -f -

# Edit ConfigMap
kubectl edit configmap $CONFIGMAP_NAME -n $NAMESPACE

# Patch specific value
kubectl patch configmap $CONFIGMAP_NAME -n $NAMESPACE \
  --type merge -p '{"data":{"log_level":"debug"}}'

# Update from file
kubectl create configmap $CONFIGMAP_NAME \
  --from-file=updated.conf \
  --dry-run=client -o yaml | \
  kubectl apply -f -
```
Updates existing ConfigMaps.

### 7. ConfigMap patterns
```bash
# Environment-specific configs
for env in dev staging prod; do
  kubectl create configmap app-config-$env \
    --from-file=config/$env/ \
    -n $env
done

# Templated ConfigMap
cat <<EOF | envsubst | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: $CONFIGMAP_NAME
  namespace: $NAMESPACE
data:
  environment: "${ENVIRONMENT}"
  region: "${AWS_REGION}"
  version: "${APP_VERSION}"
EOF

# Binary data
kubectl create configmap binary-config \
  --from-file=cert.pem \
  --from-file=image.png \
  -n $NAMESPACE
```
Common ConfigMap patterns.

### 8. Validate and troubleshoot
```bash
# View ConfigMap
kubectl get configmap $CONFIGMAP_NAME -n $NAMESPACE
kubectl describe configmap $CONFIGMAP_NAME -n $NAMESPACE

# Get ConfigMap data
kubectl get configmap $CONFIGMAP_NAME -n $NAMESPACE -o yaml

# Check specific key
kubectl get configmap $CONFIGMAP_NAME -n $NAMESPACE \
  -o jsonpath='{.data.database_host}'

# Verify mount in pod
kubectl exec pod-name -n $NAMESPACE -- ls -la /etc/config
kubectl exec pod-name -n $NAMESPACE -- cat /etc/config/app.properties

# Check environment variables
kubectl exec pod-name -n $NAMESPACE -- env | grep APP_

# Watch for changes
kubectl get configmap $CONFIGMAP_NAME -n $NAMESPACE -w
```
Validates ConfigMap usage.

## Validation
- ConfigMap created successfully
- Data keys and values are correct
- Pods can access ConfigMap data
- Mounted files have correct permissions
- Environment variables are set

## Error Handling
- **"configmap already exists"** - Delete or use `kubectl apply`
- **"key not found"** - Check ConfigMap keys with describe
- **"invalid YAML"** - Validate syntax and escaping
- **"too large"** - ConfigMaps limited to 1MB

## Safety Notes
- Don't store sensitive data (use Secrets)
- ConfigMap updates don't restart pods automatically
- Volume-mounted configs update within minutes
- Environment variables require pod restart
- Use immutable ConfigMaps in production

## Examples
- **Application configuration**
  ```bash
  kubectl create configmap app-config \
    --from-file=application.yaml \
    --from-literal=environment=production
  ```

- **Nginx configuration**
  ```bash
  kubectl create configmap nginx-config \
    --from-file=nginx.conf \
    --from-file=mime.types
  ```

- **Environment variables**
  ```bash
  kubectl create configmap env-config \
    --from-env-file=.env.production
  ```

- **Multi-file config**
  ```bash
  kubectl create configmap app-settings \
    --from-file=config/settings/ \
    --from-literal=version=1.2.3
  ```