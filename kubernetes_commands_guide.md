# Comprehensive Kubernetes (k8s) Commands Guide

## Table of Contents
1. [Most Commonly Used kubectl Commands](#most-commonly-used-kubectl-commands)
2. [Best Practices for Kubernetes Operations](#best-practices-for-kubernetes-operations)
3. [Common Kubernetes Patterns and Workflows](#common-kubernetes-patterns-and-workflows)
4. [Debugging and Troubleshooting Commands](#debugging-and-troubleshooting-commands)
5. [Resource Management Commands](#resource-management-commands)
6. [Security-Related Commands](#security-related-commands)
7. [Monitoring and Logging Commands](#monitoring-and-logging-commands)
8. [Helm Commands](#helm-commands)

## Most Commonly Used kubectl Commands

### Basic Operations

```bash
# Cluster Information
kubectl cluster-info                              # Display cluster endpoint information
kubectl get nodes                                 # List all nodes in the cluster
kubectl describe node <node-name>                 # Detailed info about a node

# Pod Management
kubectl get pods                                  # List pods in current namespace
kubectl get pods -o wide                          # List pods with additional details
kubectl get pods --all-namespaces                 # List all pods across namespaces
kubectl describe pod <pod-name>                   # Detailed pod information
kubectl logs <pod-name>                           # View pod logs
kubectl logs -f <pod-name>                        # Follow pod logs in real-time
kubectl exec -it <pod-name> -- /bin/bash         # Execute interactive shell in pod

# Deployment Management
kubectl create deployment <name> --image=<image>  # Create a deployment
kubectl get deployments                           # List deployments
kubectl scale deployment <name> --replicas=3      # Scale deployment
kubectl rollout status deployment/<name>          # Check rollout status
kubectl rollout undo deployment/<name>            # Rollback deployment

# Service Management
kubectl get services                              # List services
kubectl expose deployment <name> --port=80        # Expose deployment as service
kubectl describe service <service-name>           # Service details

# Namespace Management
kubectl get namespaces                            # List all namespaces
kubectl create namespace <name>                   # Create namespace
kubectl config set-context --current --namespace=<name>  # Switch namespace
```

### Resource Viewing and Management

```bash
# Generic resource commands
kubectl get <resource-type>                       # List resources
kubectl describe <resource-type> <name>           # Detailed resource info
kubectl delete <resource-type> <name>             # Delete resource
kubectl edit <resource-type> <name>               # Edit resource in editor

# Apply configurations
kubectl apply -f <filename.yaml>                  # Apply configuration file
kubectl apply -f <directory>                      # Apply all files in directory
kubectl delete -f <filename.yaml>                 # Delete resources from file

# Output formatting
kubectl get pods -o yaml                          # Output in YAML format
kubectl get pods -o json                          # Output in JSON format
kubectl get pods -o wide                          # Additional columns
```

## Best Practices for Kubernetes Operations

### 1. Infrastructure as Code (IaC)
- Use declarative YAML manifests for all resources
- Store configurations in version control (Git)
- Implement GitOps workflows for automated deployments

### 2. Container and Image Management
- Use specific image tags, never `latest` in production
- Optimize Docker images for minimal size
- Implement multi-stage builds
- Scan images for vulnerabilities

### 3. Resource Organization
- Use namespaces to separate environments and teams
- Apply consistent labeling strategies
- Implement resource quotas and limits

### 4. Security Best Practices
- Enable RBAC with least privilege principle
- Use Kubernetes Secrets for sensitive data
- Implement Pod Security Standards
- Regular security audits and updates

### 5. Monitoring and Observability
- Implement comprehensive logging strategy
- Set up metrics collection (Prometheus)
- Configure alerts for critical events
- Use distributed tracing for microservices

## Common Kubernetes Patterns and Workflows

### Deployment Strategies

#### 1. Rolling Update (Default)
```bash
kubectl set image deployment/<name> <container>=<new-image>
kubectl rollout status deployment/<name>
```

#### 2. Blue-Green Deployment
```yaml
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
  labels:
    version: blue

# Green deployment (new)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
  labels:
    version: green

# Switch traffic by updating service selector
kubectl patch service app-service -p '{"spec":{"selector":{"version":"green"}}}'
```

#### 3. Canary Deployment
```bash
# Deploy canary version with fewer replicas
kubectl create deployment app-canary --image=app:v2
kubectl scale deployment app-canary --replicas=1

# Gradually increase canary traffic
# Use tools like Flagger or Argo Rollouts for automation
```

## Debugging and Troubleshooting Commands

### Pod Debugging

```bash
# Basic inspection
kubectl get pod <pod-name> -o wide
kubectl describe pod <pod-name>
kubectl get events --field-selector involvedObject.name=<pod-name>

# Advanced debugging with kubectl debug
kubectl debug <pod-name> -it --image=busybox
kubectl debug <pod-name> -it --image=busybox --target=<container-name>
kubectl debug <pod-name> -it --copy-to=debug-pod --container=<container>

# Logs investigation
kubectl logs <pod-name> --previous              # Previous container logs
kubectl logs <pod-name> -c <container-name>     # Specific container logs
kubectl logs -f <pod-name> --tail=50            # Follow last 50 lines

# Execute commands in containers
kubectl exec <pod-name> -- <command>
kubectl exec -it <pod-name> -- /bin/bash
```

### Common Issues and Solutions

```bash
# Pod stuck in Pending
kubectl describe pod <pod-name>  # Check Events section

# Pod stuck in ContainerCreating
kubectl describe pod <pod-name>  # Check for image pull errors

# Pod stuck in Terminating
kubectl delete pod <pod-name> --grace-period=0 --force

# Node debugging
kubectl debug node/<node-name> -it --image=busybox
```

## Resource Management Commands

### CPU and Memory Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "1Gi"
        cpu: "500m"
```

### Resource Monitoring

```bash
# View resource usage
kubectl top nodes
kubectl top pods
kubectl top pods --containers

# Describe resource quotas
kubectl describe resourcequota

# Set resource limits for namespace
kubectl create resourcequota <name> --hard=cpu=1000,memory=200Gi
```

### Horizontal Pod Autoscaling

```bash
# Create HPA
kubectl autoscale deployment <name> --cpu-percent=50 --min=1 --max=10

# View HPA status
kubectl get hpa
kubectl describe hpa <name>
```

## Security-Related Commands

### RBAC Management

```bash
# View RBAC resources
kubectl get roles,rolebindings
kubectl get clusterroles,clusterrolebindings

# Check permissions
kubectl auth can-i create pods
kubectl auth can-i create pods --as=<username>
kubectl auth can-i create pods --as=system:serviceaccount:<namespace>:<sa-name>

# Create service account
kubectl create serviceaccount <name>

# Create role binding
kubectl create rolebinding <name> --role=<role> --user=<user>
```

### Secrets Management

```bash
# Create secrets
kubectl create secret generic <name> --from-literal=key=value
kubectl create secret generic <name> --from-file=<path>

# View secrets
kubectl get secrets
kubectl describe secret <name>

# Use secrets in pods
kubectl create secret docker-registry <name> \
  --docker-server=<server> \
  --docker-username=<user> \
  --docker-password=<password>
```

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## Monitoring and Logging Commands

### Prometheus Setup

```bash
# Install Prometheus Operator
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring

# Port forward to access Prometheus
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
```

### Grafana Access

```bash
# Get Grafana password
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

### Fluentd and Loki Integration

```bash
# Deploy Fluentd DaemonSet
kubectl apply -f https://raw.githubusercontent.com/fluent/fluentd-kubernetes-daemonset/master/fluentd-daemonset-elasticsearch-rbac.yaml

# Install Loki stack
helm install loki grafana/loki-stack -n logging --create-namespace

# Access Loki metrics
kubectl port-forward -n logging svc/loki-grafana 3000:80
```

## Helm Commands

### Basic Helm Operations

```bash
# Repository management
helm repo add <name> <url>
helm repo update
helm repo list

# Chart operations
helm search repo <chart>
helm show values <chart>

# Installation
helm install <release-name> <chart>
helm install <release-name> <chart> -f values.yaml
helm install <release-name> <chart> --set key=value

# Upgrades and rollbacks
helm upgrade <release-name> <chart>
helm rollback <release-name> <revision>

# Release management
helm list
helm status <release-name>
helm uninstall <release-name>
```

### Advanced Helm Usage

```bash
# Dry run installation
helm install <release-name> <chart> --dry-run --debug

# Create custom chart
helm create <chart-name>

# Package chart
helm package <chart-directory>

# Test installation
helm test <release-name>
```

## Quick Reference Tips

1. **Use aliases for efficiency:**
   ```bash
   alias k=kubectl
   alias kgp='kubectl get pods'
   alias kgs='kubectl get services'
   ```

2. **Enable kubectl autocompletion:**
   ```bash
   source <(kubectl completion bash)  # for bash
   source <(kubectl completion zsh)   # for zsh
   ```

3. **Use kubectl plugins:**
   ```bash
   # Install krew (plugin manager)
   kubectl krew install <plugin-name>
   ```

4. **Context switching:**
   ```bash
   kubectl config get-contexts
   kubectl config use-context <context-name>
   ```

5. **Dry run for testing:**
   ```bash
   kubectl create deployment test --image=nginx --dry-run=client -o yaml
   ```

This guide covers the essential Kubernetes commands and best practices for 2024. Remember to always test commands in non-production environments first and follow your organization's specific guidelines and policies.