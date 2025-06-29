# Minikube Image Build

## Purpose
Build and load Docker images directly into minikube's Docker daemon for local Kubernetes development

## Context
Use to build container images inside minikube without needing a registry. This speeds up development by eliminating image push/pull operations and allows testing with locally built images immediately.

## Parameters
- `$IMAGE_NAME` - Name and tag for the image
  - Required
  - Example: `myapp:latest`, `backend:v1.0`
- `$DOCKERFILE_PATH` - Path to Dockerfile
  - Optional
  - Default: `./Dockerfile`
  - Example: `./docker/Dockerfile.prod`
- `$BUILD_CONTEXT` - Docker build context path
  - Optional
  - Default: `.` (current directory)

## Steps

### 1. Configure Docker environment
```bash
# Point Docker to minikube's daemon
eval $(minikube docker-env)

# Verify configuration
docker info | grep "Name:"
# Should show minikube as the Docker host

# Show current images in minikube
docker images

# Alternative: Check minikube images
minikube image ls
```
Configures Docker to use minikube.

### 2. Build image in minikube
```bash
# Basic image build
docker build -t $IMAGE_NAME .

# Build with specific Dockerfile
docker build -t $IMAGE_NAME -f $DOCKERFILE_PATH $BUILD_CONTEXT

# Build with arguments
docker build -t $IMAGE_NAME \
  --build-arg VERSION=1.0 \
  --build-arg ENV=development \
  .

# Multi-stage build
docker build -t $IMAGE_NAME \
  --target production \
  .

# Build with no cache
docker build -t $IMAGE_NAME --no-cache .
```
Builds image inside minikube.

### 3. Load pre-built images
```bash
# Load image from tar
docker save $IMAGE_NAME -o image.tar
minikube image load image.tar

# Load from Docker Hub to minikube
minikube image pull nginx:latest

# Load local image to minikube
minikube image load $IMAGE_NAME

# Transfer from host Docker to minikube
docker save $IMAGE_NAME | (eval $(minikube docker-env) && docker load)
```
Loads existing images.

### 4. Use image in Kubernetes
```bash
# Create deployment with local image
kubectl create deployment myapp --image=$IMAGE_NAME

# Important: Set imagePullPolicy to Never
kubectl patch deployment myapp -p \
  '{"spec":{"template":{"spec":{"containers":[{"name":"myapp","imagePullPolicy":"Never"}]}}}}'

# Or create with YAML
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: $IMAGE_NAME
        imagePullPolicy: Never
EOF
```
Deploys local image.

### 5. Development workflow
```bash
# Function for quick rebuild and deploy
rebuild_and_deploy() {
  local app_name=$1
  local image_name=$2
  
  # Ensure using minikube docker
  eval $(minikube docker-env)
  
  # Build new image
  docker build -t $image_name .
  
  # Delete existing pods to force refresh
  kubectl delete pods -l app=$app_name
  
  # Wait for new pods
  kubectl wait --for=condition=ready pod -l app=$app_name --timeout=60s
}

# Watch for file changes and rebuild
while inotifywait -r -e modify,create,delete ./src; do
  rebuild_and_deploy myapp myapp:latest
done
```
Automates development cycle.

### 6. Multi-architecture builds
```bash
# Build for minikube's architecture
ARCH=$(minikube ssh -- uname -m)
docker build -t $IMAGE_NAME --platform=linux/$ARCH .

# Build multi-platform image
docker buildx create --use --name minikube-builder
docker buildx build \
  --platform=linux/amd64,linux/arm64 \
  -t $IMAGE_NAME \
  --load \
  .
```
Handles architecture compatibility.

### 7. Cache and optimize builds
```bash
# Pre-pull base images
minikube image pull node:16-alpine
minikube image pull python:3.9-slim

# Cache dependencies
cat <<EOF > Dockerfile.cached
FROM node:16-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:16-alpine
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]
EOF

docker build -t myapp:cached -f Dockerfile.cached .

# Use BuildKit for better caching
DOCKER_BUILDKIT=1 docker build -t $IMAGE_NAME .
```
Optimizes build performance.

### 8. Troubleshooting and cleanup
```bash
# Check if image exists in minikube
minikube ssh -- docker images | grep $IMAGE_NAME

# Debug build issues
docker build -t $IMAGE_NAME . --progress=plain

# Clean up unused images
docker image prune -a

# Remove specific image
docker rmi $IMAGE_NAME

# Reset Docker environment
eval $(minikube docker-env -u)

# Full cleanup
minikube ssh -- docker system prune -a

# Verify image is accessible
kubectl run test --image=$IMAGE_NAME --image-pull-policy=Never --rm -it -- sh
```
Handles common issues.

## Validation
- Image appears in `docker images` output
- Image listed in `minikube image ls`
- Pods can use image with `imagePullPolicy: Never`
- No `ImagePullBackOff` errors
- Application runs correctly

## Error Handling
- **"Cannot connect to Docker daemon"** - Run `eval $(minikube docker-env)`
- **"ImagePullBackOff"** - Set `imagePullPolicy: Never`
- **"No space left"** - Clean up old images or increase disk size
- **"Build failed"** - Check Dockerfile and build context

## Safety Notes
- Always use `imagePullPolicy: Never` for local images
- Clean up old images regularly
- Remember to reset Docker env when done
- Tag images properly to avoid conflicts
- Test builds before deploying

## Examples
- **Simple build and deploy**
  ```bash
  eval $(minikube docker-env)
  docker build -t myapp:local .
  kubectl create deployment myapp --image=myapp:local
  kubectl set image deployment/myapp myapp=myapp:local --record
  ```

- **Development iteration**
  ```bash
  eval $(minikube docker-env)
  docker build -t backend:dev .
  kubectl rollout restart deployment/backend
  ```

- **Load from registry**
  ```bash
  minikube image pull gcr.io/myproject/app:latest
  kubectl create deployment app --image=gcr.io/myproject/app:latest
  ```

- **Multi-stage build**
  ```bash
  eval $(minikube docker-env)
  docker build -t frontend:prod --target=production -f Dockerfile.multi .
  ```