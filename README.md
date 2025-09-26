# WordPress ArgoCD Deployment Helm Values

This repository contains Helm values for deploying WordPress using the Bitnami WordPress chart with ArgoCD.

## Upload Size Configuration

The WordPress deployment has been configured with increased upload size limits:

- **Upload max filesize**: 128M
- **Post max size**: 128M 
- **Memory limit**: 256M
- **Max execution time**: 300 seconds
- **Max input time**: 300 seconds
- **Nginx proxy body size**: 128m

These settings allow for larger file uploads through both WordPress and the ingress controller.

## Configuration Details

The upload size limits are configured through:

1. **PHP Settings** - Set via `wordpressExtraConfigContent` in values.yaml:
   ```yaml
   wordpressExtraConfigContent: |
     @ini_set('upload_max_filesize', '128M');
     @ini_set('post_max_size', '128M');
     @ini_set('memory_limit', '256M');
     @ini_set('max_execution_time', '300');
     @ini_set('max_input_time', '300');
   ```

2. **Ingress Settings** - Set via ingress annotations in values.yaml:
   ```yaml
   ingress:
     annotations:
       nginx.ingress.kubernetes.io/proxy-body-size: "128m"
   ```

## Usage

Deploy with ArgoCD or apply directly with Helm:

```bash
helm upgrade --install wordpress bitnami/wordpress -f values.yaml
```