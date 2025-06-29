# K8s Secret Manage

## Purpose
Create, update, and manage Kubernetes secrets for storing sensitive data like passwords, tokens, and keys

## Context
Use secrets to store and manage sensitive information separately from application code. Secrets are base64 encoded (not encrypted) by default and can be mounted as files or exposed as environment variables.

## Parameters
- `$SECRET_NAME` - Name of the secret
  - Required
  - Example: `app-secrets`, `db-credentials`
- `$SECRET_TYPE` - Type of secret to create
  - Optional
  - Default: `generic`
  - Options: `generic`, `docker-registry`, `tls`
- `$NAMESPACE` - Target namespace
  - Optional
  - Default: `default`

## Steps

### 1. Create generic secret
```bash
# From literal values
kubectl create secret generic $SECRET_NAME \
  --from-literal=username=admin \
  --from-literal=password='S3cur3P@ssw0rd' \
  --from-literal=api-key='abc123xyz789' \
  -n $NAMESPACE

# From files
kubectl create secret generic $SECRET_NAME \
  --from-file=ssh-privatekey=/path/to/.ssh/id_rsa \
  --from-file=ssh-publickey=/path/to/.ssh/id_rsa.pub \
  -n $NAMESPACE

# From env file
kubectl create secret generic $SECRET_NAME \
  --from-env-file=path/to/.env \
  -n $NAMESPACE
```
Creates generic secrets.

### 2. Create Docker registry secret
```bash
# Docker Hub
kubectl create secret docker-registry $SECRET_NAME \
  --docker-server=docker.io \
  --docker-username=$DOCKER_USER \
  --docker-password=$DOCKER_PASSWORD \
  --docker-email=$DOCKER_EMAIL \
  -n $NAMESPACE

# Private registry
kubectl create secret docker-registry $SECRET_NAME \
  --docker-server=$REGISTRY_URL \
  --docker-username=$REGISTRY_USER \
  --docker-password=$REGISTRY_PASSWORD \
  -n $NAMESPACE

# From existing Docker config
kubectl create secret generic $SECRET_NAME \
  --from-file=.dockerconfigjson=$HOME/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson \
  -n $NAMESPACE
```
Creates registry authentication secrets.

### 3. Create TLS secret
```bash
# From certificate files
kubectl create secret tls $SECRET_NAME \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key \
  -n $NAMESPACE

# Generate self-signed cert and create secret
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=example.com/O=example"

kubectl create secret tls $SECRET_NAME \
  --cert=tls.crt \
  --key=tls.key \
  -n $NAMESPACE
```
Creates TLS certificate secrets.

### 4. Create secret from YAML
```bash
# Encode values
echo -n 'admin' | base64
echo -n 'password123' | base64

# Create secret manifest
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: $SECRET_NAME
  namespace: $NAMESPACE
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQxMjM=
stringData:
  plain-text: "This will be encoded automatically"
EOF
```
Creates secrets using manifests.

### 5. Update existing secrets
```bash
# Add new key to existing secret
kubectl get secret $SECRET_NAME -n $NAMESPACE -o json | \
  jq '.data["new-key"]="'$(echo -n 'new-value' | base64)'"' | \
  kubectl apply -f -

# Replace entire secret
kubectl create secret generic $SECRET_NAME \
  --from-literal=key1=value1 \
  --from-literal=key2=value2 \
  --dry-run=client -o yaml | \
  kubectl replace -f -

# Patch specific field
kubectl patch secret $SECRET_NAME -n $NAMESPACE \
  --type='json' \
  -p='[{"op": "replace", "path": "/data/password", "value":"'$(echo -n 'newpass' | base64)'"}]'
```
Updates secret values.

### 6. Use secrets in pods
```bash
# Mount as volume
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
  namespace: $NAMESPACE
spec:
  containers:
  - name: test-container
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: $SECRET_NAME
      defaultMode: 0400
EOF

# Use as environment variables
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
  namespace: $NAMESPACE
spec:
  containers:
  - name: test-container
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: $SECRET_NAME
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: $SECRET_NAME
          key: password
EOF
```
Configures pods to use secrets.

### 7. Manage secret access
```bash
# Create ServiceAccount with imagePullSecrets
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: $NAMESPACE
imagePullSecrets:
- name: $SECRET_NAME
EOF

# RBAC for secret access
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: $NAMESPACE
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["$SECRET_NAME"]
  verbs: ["get", "list"]
EOF
```
Controls secret access.

### 8. Secret operations and validation
```bash
# List secrets
kubectl get secrets -n $NAMESPACE

# Describe secret (doesn't show values)
kubectl describe secret $SECRET_NAME -n $NAMESPACE

# Get secret data (base64 encoded)
kubectl get secret $SECRET_NAME -n $NAMESPACE -o jsonpath='{.data}'

# Decode secret value
kubectl get secret $SECRET_NAME -n $NAMESPACE \
  -o jsonpath='{.data.password}' | base64 -d

# Export secret to file
kubectl get secret $SECRET_NAME -n $NAMESPACE \
  -o jsonpath='{.data.ssh-privatekey}' | base64 -d > id_rsa

# Copy secret to another namespace
kubectl get secret $SECRET_NAME -n $NAMESPACE -o yaml | \
  sed "s/namespace: $NAMESPACE/namespace: $NEW_NAMESPACE/" | \
  kubectl apply -f -

# Delete secret
kubectl delete secret $SECRET_NAME -n $NAMESPACE
```
Common secret operations.

## Validation
- Secret created successfully
- Base64 encoding is correct
- Pods can access secret data
- Permissions are properly restricted
- No sensitive data in logs

## Error Handling
- **"secret already exists"** - Use `kubectl replace` or delete first
- **"invalid base64"** - Ensure proper encoding
- **"key not found"** - Check secret keys with `kubectl describe`
- **"permission denied"** - Verify RBAC and ServiceAccount

## Safety Notes
- Never commit secrets to version control
- Use RBAC to limit secret access
- Enable encryption at rest in cluster
- Rotate secrets regularly
- Audit secret access
- Consider using external secret managers

## Examples
- **Database credentials**
  ```bash
  kubectl create secret generic db-secret \
    --from-literal=host=postgres.example.com \
    --from-literal=username=dbuser \
    --from-literal=password='$(openssl rand -base64 32)'
  ```

- **TLS certificate**
  ```bash
  kubectl create secret tls ingress-tls \
    --cert=./tls.crt --key=./tls.key \
    -n ingress-nginx
  ```

- **Docker registry**
  ```bash
  kubectl create secret docker-registry gcr-secret \
    --docker-server=gcr.io \
    --docker-username=_json_key \
    --docker-password="$(cat gcr-key.json)"
  ```

- **SSH keys**
  ```bash
  kubectl create secret generic ssh-key-secret \
    --from-file=id_rsa=$HOME/.ssh/id_rsa \
    --from-file=known_hosts=$HOME/.ssh/known_hosts
  ```