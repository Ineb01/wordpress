# WordPress ArgoCD Deployment

This repository contains a complete ArgoCD deployment configuration for WordPress with MySQL backend, including all necessary Kubernetes manifests and ArgoCD Application definitions.

## Architecture

The deployment consists of:

- **WordPress**: Latest WordPress 6.4 with Apache (configured for multisite subdomain mode)
- **MySQL**: MySQL 8.0 database backend
- **Persistent Storage**: Separate PVCs for WordPress and MySQL data
- **ConfigMaps & Secrets**: Secure configuration management
- **Ingress**: Multiple subdomain support for multisite network

## Multisite Configuration

This WordPress deployment is configured as a **multisite network in subdomain mode** with the following domains:

- **bella-margherita.dphx.eu** - Pizzeria site
- **gentlmens-cut.dphx.eu** - Barber shop site

### Multisite Features

- **Subdomain Install**: Each site gets its own subdomain
- **Shared Resources**: All sites share the same WordPress codebase and database
- **Centralized Management**: Manage all sites from the network admin dashboard
- **Wildcard SSL**: Supports `*.dphx.eu` wildcard certificate via Let's Encrypt

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

Once deployed, the WordPress multisite network will be available at:

- **Bella Margherita Pizzeria**: https://bella-margherita.dphx.eu
- **Gentlemen's Cut Barber Shop**: https://gentlmens-cut.dphx.eu

### Initial Setup

1. Visit one of the domains to complete the WordPress installation
2. After initial setup, access the **Network Admin** at `/wp-admin/network/`
3. From Network Admin, you can:
   - Add new sites to the network
   - Manage existing sites
   - Install themes and plugins network-wide
   - Configure network settings

### Adding More Sites

To add additional subdomains to the multisite network:

1. Update the ingress configuration in `manifests/wordpress/ingress.yaml`
2. Add the new subdomain to the `hosts` list under `tls` and create a new rule
3. Apply the changes: `kubectl apply -k manifests/`
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

### Upload Limits

The WordPress deployment is configured with increased file upload limits:

- **Upload Max File Size**: 500MB
- **Post Max Size**: 512MB  
- **Memory Limit**: 512MB
- **Max Execution Time**: 300 seconds

These settings are configured via a custom PHP configuration file (`manifests/wordpress/php-config.yaml`) that is mounted into the WordPress container at `/usr/local/etc/php/conf.d/uploads.ini`.

### Apache Configuration

The deployment includes a custom Apache configuration (`manifests/wordpress/apache-config.yaml`) that:

- **Eliminates ServerName Warning**: Sets `ServerName wordpress.cluster.dphx.eu` to suppress the Apache warning `AH00558: Could not reliably determine the server's fully qualified domain name`
- **WordPress Directory Permissions**: Configures proper `AllowOverride All` settings for WordPress functionality
- **Logging Configuration**: Sets up standard Apache error and access logging

The configuration is mounted to `/etc/apache2/sites-available/000-default.conf` in the WordPress container.

## Troubleshooting

### Common Issues

1. **Pods in Pending State**: Check PVC status and storage class availability
2. **Database Connection Issues**: Verify MySQL pod is running and credentials match
3. **External Access Issues**: Check LoadBalancer configuration and firewall rules
4. **Apache ServerName Warning**: The deployment includes a custom Apache configuration to suppress the `AH00558` warning about determining the server's FQDN

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
