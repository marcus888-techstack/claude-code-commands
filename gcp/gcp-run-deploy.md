# GCP Run Deploy

## Purpose
Deploy a containerized application to Google Cloud Run serverless platform

## Context
Use to deploy stateless HTTP-based services that automatically scale. Cloud Run handles infrastructure, scaling from zero to thousands of instances based on traffic. Ideal for web apps, APIs, and microservices.

## Parameters
- `$SERVICE_NAME` - Name for the Cloud Run service
  - Required
  - Example: `my-api`, `web-frontend`
- `$IMAGE` - Container image to deploy
  - Required
  - Example: `gcr.io/my-project/app:latest`, `us-docker.pkg.dev/my-project/repo/image:v1`
- `$REGION` - Region to deploy to
  - Optional
  - Default: `us-central1`
  - Example: `us-east1`, `europe-west1`
- `$OPTIONS` - Additional deployment options
  - Optional
  - Example: `--port=8080`, `--memory=512Mi`, `--allow-unauthenticated`

## Steps

### 1. Check Cloud Run API
```bash
if ! gcloud services list --enabled --filter="name:run.googleapis.com" --format="value(name)" | grep -q run; then
    echo "Enabling Cloud Run API..."
    gcloud services enable run.googleapis.com
    sleep 5
fi
```
Ensures Cloud Run API is enabled.

### 2. Set deployment parameters
```bash
REGION=${REGION:-us-central1}
PROJECT=$(gcloud config get-value project)

echo "=== Deployment Configuration ==="
echo "Service: $SERVICE_NAME"
echo "Image: $IMAGE"
echo "Region: $REGION"
echo "Project: $PROJECT"
```
Configures deployment parameters.

### 3. Validate image exists
```bash
echo -e "\n=== Validating Image ==="
# Check if using Artifact Registry
if [[ "$IMAGE" =~ .*pkg.dev.* ]]; then
    REGISTRY_LOCATION=$(echo $IMAGE | cut -d'/' -f1 | cut -d'-' -f1)
    if ! gcloud artifacts docker images list $(dirname $IMAGE) --include-tags 2>/dev/null | grep -q $(basename $IMAGE); then
        echo "⚠️  Warning: Image may not exist in Artifact Registry"
        echo "Push image with: docker push $IMAGE"
    else
        echo "✓ Image found in Artifact Registry"
    fi
# Check if using Container Registry
elif [[ "$IMAGE" =~ ^gcr.io.* ]]; then
    if ! gcloud container images describe $IMAGE &>/dev/null; then
        echo "⚠️  Warning: Image may not exist in Container Registry"
        echo "Push image with: docker push $IMAGE"
    else
        echo "✓ Image found in Container Registry"
    fi
fi
```
Verifies the container image is available.

### 4. Check for existing service
```bash
echo -e "\n=== Checking Existing Service ==="
if gcloud run services describe $SERVICE_NAME --region=$REGION &>/dev/null; then
    echo "Service already exists - will update"
    EXISTING_URL=$(gcloud run services describe $SERVICE_NAME --region=$REGION --format="value(status.url)")
    echo "Current URL: $EXISTING_URL"
else
    echo "Creating new service"
fi
```
Determines if this is a new deployment or update.

### 5. Deploy the service
```bash
echo -e "\n=== Deploying Service ==="
gcloud run deploy $SERVICE_NAME \
    --image=$IMAGE \
    --region=$REGION \
    --platform=managed \
    $OPTIONS
```
Deploys the service to Cloud Run.

### 6. Wait for deployment completion
```bash
echo -e "\n=== Deployment Status ==="
for i in {1..30}; do
    STATUS=$(gcloud run services describe $SERVICE_NAME --region=$REGION --format="value(status.conditions[0].status)" 2>/dev/null)
    
    if [ "$STATUS" = "True" ]; then
        echo "✓ Deployment successful"
        break
    else
        echo -ne "\rDeploying... ($i/30)"
        sleep 2
    fi
    
    if [ $i -eq 30 ]; then
        echo "\n⚠️  Deployment may still be in progress"
    fi
done
```
Monitors deployment progress.

### 7. Display service information
```bash
echo -e "\n=== Service Details ==="
gcloud run services describe $SERVICE_NAME --region=$REGION --format="yaml(
    metadata.name,
    spec.template.spec.containers[0].image,
    spec.template.spec.containers[0].resources,
    spec.template.metadata.annotations
)"
```
Shows detailed service configuration.

### 8. Get service URL and test
```bash
echo -e "\n=== Service Endpoint ==="
SERVICE_URL=$(gcloud run services describe $SERVICE_NAME --region=$REGION --format="value(status.url)")
echo "Service URL: $SERVICE_URL"

# Test if service is accessible
if [[ "$OPTIONS" == *"--allow-unauthenticated"* ]]; then
    echo -e "\nTesting endpoint..."
    if curl -s -o /dev/null -w "%{http_code}" $SERVICE_URL | grep -q "200\|301\|302"; then
        echo "✓ Service is responding"
    else
        echo "⚠️  Service may not be ready yet"
    fi
else
    echo -e "\nNote: Service requires authentication"
    echo "To make public: gcloud run services add-iam-policy-binding $SERVICE_NAME --region=$REGION --member='allUsers' --role='roles/run.invoker'"
fi
```
Provides service URL and tests accessibility.

### 9. Display traffic and revisions
```bash
echo -e "\n=== Traffic Allocation ==="
gcloud run services describe $SERVICE_NAME --region=$REGION --format="table(
    spec.traffic[].revisionName,
    spec.traffic[].percent
)"

echo -e "\n=== Recent Revisions ==="
gcloud run revisions list --service=$SERVICE_NAME --region=$REGION --limit=3 --format="table(
    metadata.name,
    metadata.creationTimestamp,
    spec.containers[0].image
)"
```
Shows traffic distribution and revision history.

### 10. Provide management commands
```bash
echo -e "\n=== Management Commands ==="
cat << EOF
# View logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=$SERVICE_NAME" --limit=50

# Update traffic split
gcloud run services update-traffic $SERVICE_NAME --region=$REGION --to-latest

# Delete service
gcloud run services delete $SERVICE_NAME --region=$REGION

# Set environment variables
gcloud run services update $SERVICE_NAME --region=$REGION --set-env-vars KEY=VALUE

# Update memory/CPU
gcloud run services update $SERVICE_NAME --region=$REGION --memory=1Gi --cpu=2
EOF
```
Provides useful management commands.

## Validation
- Service is deployed successfully
- Service URL is accessible
- Container image is running
- Health checks pass

## Error Handling
- **"Image not found"** - Verify image path and push to registry
- **"Permission denied"** - Need run.services.create permission
- **"Invalid configuration"** - Check memory, CPU, and port settings
- **"Quota exceeded"** - Check Cloud Run quotas for region
- **"Build failed"** - If using source deployment, check build logs

## Safety Notes
- Deployed services incur costs based on usage
- Default allows unauthenticated access - secure appropriately
- Environment variables are visible in console
- Use Secret Manager for sensitive values
- Set appropriate memory and CPU limits

## Examples
- **Deploy public API**
  ```
  gcp-run-deploy my-api gcr.io/my-project/api:v1 us-central1 "--allow-unauthenticated --port=8080"
  ```
  Deploys publicly accessible API

- **Deploy authenticated service**
  ```
  gcp-run-deploy internal-app us-docker.pkg.dev/project/repo/app:latest
  ```
  Deploys service requiring authentication

- **Deploy with environment variables**
  ```
  gcp-run-deploy web-app gcr.io/project/app:prod us-east1 "--set-env-vars=NODE_ENV=production,API_KEY=xyz"
  ```
  Deploys with environment configuration

- **Deploy with custom resources**
  ```
  gcp-run-deploy heavy-service gcr.io/project/service:latest europe-west1 "--memory=2Gi --cpu=2"
  ```
  Deploys with increased resources