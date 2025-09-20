# WordPress ArgoCD Deployment

This repository contains a complete ArgoCD deployment configuration for WordPress with MySQL backend, including all necessary Kubernetes manifests and ArgoCD Application definitions.

## Architecture

The deployment consists of:

- **WordPress**: Latest WordPress 6.4 with Apache
- **MySQL**: MySQL 8.0 database backend
- **Persistent Storage**: Separate PVCs for WordPress and MySQL data
- **ConfigMaps & Secrets**: Secure configuration management
- **LoadBalancer Service**: External access to WordPress

## Repository Structure

```
├── README.md
├── argocd/
│   └── application.yaml       # ArgoCD Application definition
└── manifests/
    ├── namespace.yaml         # WordPress namespace
    ├── kustomization.yaml     # Kustomize configuration
    ├── wordpress/
    │   ├── deployment.yaml    # WordPress deployment
    │   ├── service.yaml       # WordPress LoadBalancer service
    │   ├── pvc.yaml          # WordPress persistent storage
    │   └── configmap.yaml    # WordPress configuration
    └── mysql/
        ├── deployment.yaml    # MySQL deployment
        ├── service.yaml       # MySQL ClusterIP service
        ├── secret.yaml        # MySQL credentials
        └── pvc.yaml          # MySQL persistent storage
```

## Prerequisites

- ArgoCD installed and running in your Kubernetes cluster
- Kubernetes cluster with persistent volume support
- LoadBalancer support (cloud provider or MetalLB for on-premise)

## Deployment Instructions

### Option 1: Deploy via ArgoCD UI

1. Access your ArgoCD UI
2. Click "New App"
3. Fill in the application details:
   - **Application Name**: `wordpress`
   - **Project**: `default`
   - **Repository URL**: `https://github.com/Ineb01/wordpress`
   - **Path**: `manifests`
   - **Destination Server**: `https://kubernetes.default.svc`
   - **Namespace**: `wordpress`
4. Enable **Auto-sync** and **Self Heal**
5. Click "Create"

### Option 2: Deploy via kubectl

```bash
# Apply the ArgoCD Application
kubectl apply -f argocd/application.yaml

# Wait for the application to sync
kubectl get applications -n argocd
```

### Option 3: Manual Deployment (without ArgoCD)

```bash
# Deploy all manifests directly
kubectl apply -k manifests/
```

## Configuration

### Database Credentials

The MySQL credentials are stored in `manifests/mysql/secret.yaml` with base64 encoded values:

- **Root Password**: `rootpassword`
- **Database**: `wordpress`
- **Username**: `wordpress`
- **Password**: `wordpresspass`

**⚠️ Security Note**: Change these default credentials before production deployment!

### Storage

- **WordPress PVC**: 20Gi for WordPress files and uploads
- **MySQL PVC**: 10Gi for database storage

### Resources

- **WordPress**: 256Mi-512Mi RAM, 200m-400m CPU
- **MySQL**: 512Mi-1Gi RAM, 250m-500m CPU

## Accessing WordPress

Once deployed, WordPress will be available via the LoadBalancer service:

```bash
# Get the external IP
kubectl get service wordpress-service -n wordpress

# If using port-forward for testing
kubectl port-forward service/wordpress-service 8080:80 -n wordpress
```

Access WordPress at:
- LoadBalancer: `http://<EXTERNAL-IP>`
- Port-forward: `http://localhost:8080`

## Monitoring Deployment

```bash
# Check ArgoCD application status
kubectl get applications wordpress -n argocd

# Check pods status
kubectl get pods -n wordpress

# Check services
kubectl get services -n wordpress

# Check persistent volumes
kubectl get pvc -n wordpress
```

## Customization

### Updating WordPress/MySQL Versions

Edit the image tags in `manifests/kustomization.yaml`:

```yaml
images:
  - name: wordpress
    newTag: "6.5-apache"  # Update version
  - name: mysql
    newTag: "8.1"         # Update version
```

### Scaling

Update replicas in the deployment files:

```yaml
spec:
  replicas: 2  # Add this line to scale WordPress
```

### Resource Limits

Modify the `resources` section in deployment files to adjust CPU/memory limits.

## Troubleshooting

### Common Issues

1. **Pods in Pending State**: Check PVC status and storage class availability
2. **Database Connection Issues**: Verify MySQL pod is running and credentials match
3. **External Access Issues**: Check LoadBalancer configuration and firewall rules

### Useful Commands

```bash
# View logs
kubectl logs -l app=wordpress -n wordpress
kubectl logs -l app=mysql -n wordpress

# Check events
kubectl get events -n wordpress --sort-by='.lastTimestamp'

# Describe resources
kubectl describe deployment wordpress -n wordpress
kubectl describe deployment mysql -n wordpress
```

## Security Considerations

- Change default database passwords before production use
- Consider using external database services for production
- Implement network policies for enhanced security
- Use HTTPS with proper SSL certificates
- Regular backup of persistent volumes
- Keep WordPress and MySQL images updated

## Production Recommendations

- Use managed database services (RDS, Cloud SQL, etc.)
- Implement proper backup strategies
- Configure resource monitoring and alerting
- Use secrets management solutions (External Secrets, Vault)
- Implement proper RBAC and network policies
- Use ingress controllers with SSL termination
- Configure horizontal pod autoscaling
