# Kubernetes Commands Registry

## Overview
Comprehensive collection of Kubernetes (k8s) and minikube commands for container orchestration, deployment, and management.

## Core Kubernetes Commands

### Debugging & Troubleshooting
- **[k8s-pod-debug](./k8s-pod-debug.md)** - Debug problematic pods with logs, exec, and events
- **[k8s-logs-view](./k8s-logs-view.md)** - View and analyze logs from pods and deployments

### Deployment Management  
- **[k8s-deployment-rollout](./k8s-deployment-rollout.md)** - Perform rolling updates with zero downtime
- **[k8s-scale-replicas](./k8s-scale-replicas.md)** - Scale deployments to handle load changes
- **[k8s-helm-deploy](./k8s-helm-deploy.md)** - Deploy applications using Helm charts

### Networking & Services
- **[k8s-service-expose](./k8s-service-expose.md)** - Expose deployments as services for network access

### Configuration & Secrets
- **[k8s-configmap-create](./k8s-configmap-create.md)** - Create ConfigMaps for application configuration
- **[k8s-secret-manage](./k8s-secret-manage.md)** - Manage sensitive data with Kubernetes secrets

### Cluster Organization
- **[k8s-namespace-create](./k8s-namespace-create.md)** - Create namespaces for resource isolation

## Minikube Commands

### Local Development
- **[minikube-start-cluster](./minikube-start-cluster.md)** - Start local Kubernetes cluster for development
- **[minikube-image-build](./minikube-image-build.md)** - Build Docker images directly in minikube

## Command Categories

### 🔍 Debugging & Monitoring
Commands for troubleshooting and observing cluster state:
- Pod debugging with logs and exec
- Real-time log streaming
- Event monitoring
- Resource usage tracking

### 🚀 Deployment & Scaling
Commands for application lifecycle management:
- Rolling updates and rollbacks
- Horizontal scaling
- Helm chart deployments
- Canary and blue-green deployments

### 🔌 Networking
Commands for service exposure and connectivity:
- ClusterIP, NodePort, and LoadBalancer services
- Ingress configuration
- Network policies
- Service discovery

### 🔐 Configuration & Security
Commands for managing application settings:
- ConfigMaps for non-sensitive data
- Secrets for sensitive information
- RBAC configuration
- Security contexts

### 🏗️ Development Environment
Commands for local Kubernetes development:
- Minikube cluster management
- Local image building
- Volume mounting
- Addon configuration

## Quick Start Examples

### Deploy an Application
```bash
# Start minikube
minikube start --cpus=4 --memory=8192

# Build image locally
eval $(minikube docker-env)
docker build -t myapp:local .

# Create deployment
kubectl create deployment myapp --image=myapp:local
kubectl set image deployment/myapp myapp=myapp:local

# Expose service
kubectl expose deployment myapp --port=80 --target-port=8080 --type=NodePort

# Get URL
minikube service myapp --url
```

### Debug a Pod
```bash
# Check pod status
kubectl get pods
kubectl describe pod <pod-name>

# View logs
kubectl logs -f <pod-name>

# Execute commands
kubectl exec -it <pod-name> -- /bin/bash
```

### Scale Application
```bash
# Manual scaling
kubectl scale deployment myapp --replicas=5

# Autoscaling
kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=80
```

## Best Practices

### Resource Management
- Always set resource requests and limits
- Use namespaces for environment separation
- Implement resource quotas
- Monitor resource usage

### Security
- Use secrets for sensitive data
- Implement RBAC policies
- Enable network policies
- Scan images for vulnerabilities

### Deployment
- Use health checks (liveness/readiness probes)
- Implement graceful shutdown
- Use rolling update strategies
- Maintain deployment history

### Development
- Use minikube profiles for different projects
- Clean up unused resources regularly
- Use consistent labeling
- Document your deployments

## Common Workflows

### Development Cycle
1. Start minikube cluster
2. Build image in minikube
3. Deploy to cluster
4. Test and debug
5. Iterate with hot reload

### Production Deployment
1. Create namespace
2. Set up ConfigMaps/Secrets
3. Deploy with Helm
4. Configure autoscaling
5. Monitor and maintain

### Troubleshooting Flow
1. Check pod status
2. Examine events
3. Review logs
4. Debug with exec
5. Check resource constraints

## Additional Resources

### Official Documentation
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [kubectl Reference](https://kubernetes.io/docs/reference/kubectl/)
- [Minikube Docs](https://minikube.sigs.k8s.io/docs/)
- [Helm Docs](https://helm.sh/docs/)

### Useful Tools
- **k9s** - Terminal UI for Kubernetes
- **kubectx/kubens** - Context and namespace switching
- **stern** - Multi-pod log tailing
- **kustomize** - Template-free configuration

### Learning Resources
- Kubernetes the Hard Way
- Official Kubernetes tutorials
- Cloud provider K8s services docs
- Container orchestration patterns