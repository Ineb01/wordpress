# WordPress Multisite ArgoCD Deployment

This repository contains a complete ArgoCD deployment configuration for WordPress Multisite with MySQL backend, including all necessary Kubernetes manifests and ArgoCD Application definitions.

## Architecture

The deployment consists of:

- **WordPress Multisite**: Latest WordPress 6.4 with Apache configured for multisite network
- **MySQL**: MySQL 8.0 database backend
- **Persistent Storage**: Separate PVCs for WordPress and MySQL data
- **ConfigMaps & Secrets**: Secure configuration management
- **Ingress Controller**: External access via multiple domains with TLS termination

## Multisite Configuration

This deployment is configured as a WordPress multisite network with the following domains:

- **Primary Site**: `bella-margherita.dphx.eu` (Pizzeria)
- **Secondary Site**: `gentlmens-cut.dphx.eu` (Barber Shop)

Both sites are managed through a single WordPress installation with subdirectory-based multisite configuration.

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

Once deployed, WordPress multisite will be available via the ingress controller at:

- **Pizzeria Site**: `https://bella-margherita.dphx.eu`
- **Barber Shop Site**: `https://gentlmens-cut.dphx.eu`

Both domains are secured with automatic TLS certificates via cert-manager and Let's Encrypt.

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

### Upload Limits

The WordPress deployment is configured with increased file upload limits:

- **Upload Max File Size**: 500MB
- **Post Max Size**: 512MB  
- **Memory Limit**: 512MB
- **Max Execution Time**: 300 seconds

These settings are configured via a custom PHP configuration file (`manifests/wordpress/php-config.yaml`) that is mounted into the WordPress container at `/usr/local/etc/php/conf.d/uploads.ini`.

### Multisite Network Setup

After the initial deployment, you'll need to complete the multisite network setup through the WordPress admin interface:

1. Access the primary site at `https://bella-margherita.dphx.eu/wp-admin`
2. Complete the initial WordPress setup
3. Go to **Tools > Network Setup** to configure the multisite network
4. Add the secondary site for the barber shop:
   - Navigate to **My Sites > Network Admin > Sites**
   - Click **Add New** and enter `gentlmens-cut.dphx.eu` as the site address
   - Configure the site title and admin email

The multisite configuration is pre-configured in the deployment with:
- Network type: Subdirectory (not subdomain)
- Primary domain: `bella-margherita.dphx.eu`
- Network allows both domains to work correctly

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
