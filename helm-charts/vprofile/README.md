# Vprofile Helm Chart

This Helm chart deploys the vprofile multi-tier application on Kubernetes.

## Components

- **Java Application (vproapp)**: Tomcat-based Java application
- **MySQL Database (vprodb)**: Backend database with persistent storage
- **Memcached (vpromc)**: Caching layer
- **RabbitMQ (vprormq)**: Message broker
- **Nginx (vproginx)**: Optional web server (disabled by default)

## Prerequisites

- Kubernetes cluster (1.19+)
- Helm 3.0+
- Ingress controller (nginx-ingress)
- kubectl configured

## Installation

### 1. Install Nginx Ingress Controller (if not already installed)

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
```

### 2. Install vprofile Application

```bash
# From the repository root directory
cd helm-charts

# Install with default values
helm install vprofile ./vprofile

# Install with custom namespace
helm install vprofile ./vprofile -n vprofile --create-namespace

# Install with custom values
helm install vprofile ./vprofile -f custom-values.yaml
```

### 3. Verify Installation

```bash
# Check release
helm list

# Check status
helm status vprofile

# Check all resources
kubectl get all

# Check ingress
kubectl get ingress
```

## Configuration

The following table lists the configurable parameters and their default values.

### Application Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `app.name` | Application name | `vproapp` |
| `app.replicas` | Number of replicas | `2` |
| `app.image` | Docker image | `vprocontainers/vprofileapp` |
| `app.port` | Application port | `8080` |

### Database Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `db.name` | Database name | `vprodb` |
| `db.image` | Docker image | `nkposa/vprofiledb:latest` |
| `db.port` | Database port | `3306` |
| `db.storage.size` | PVC storage size | `1Gi` |
| `db.storage.storageClass` | Storage class | `gp2` |

### Ingress Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `true` |
| `ingress.className` | Ingress class | `nginx` |
| `ingress.hosts` | Host configurations | See values.yaml |

## Upgrading

```bash
# Upgrade with new values
helm upgrade vprofile ./vprofile

# Upgrade with custom values file
helm upgrade vprofile ./vprofile -f custom-values.yaml

# Upgrade and force recreation
helm upgrade vprofile ./vprofile --force
```

## Rollback

```bash
# List revisions
helm history vprofile

# Rollback to previous version
helm rollback vprofile

# Rollback to specific revision
helm rollback vprofile 1
```

## Uninstallation

```bash
# Uninstall the release
helm uninstall vprofile

# Uninstall and delete namespace (if created)
helm uninstall vprofile -n vprofile
kubectl delete namespace vprofile

# Delete ingress controller (if needed)
helm uninstall ingress-nginx -n ingress-nginx
kubectl delete namespace ingress-nginx
```

## Customization

### Enable/Disable Nginx

```yaml
nginx:
  enabled: false  # Set to false to disable nginx
```

### Change Replica Count

```yaml
app:
  replicas: 3  # Scale application to 3 replicas
```

### Change Storage Size

```yaml
db:
  storage:
    size: 5Gi  # Increase database storage to 5Gi
```

### Add More Ingress Hosts

```yaml
ingress:
  hosts:
    - host: vprofile.posafoods.online
    - host: hima.posafoods.online
    - host: new-domain.example.com
```

## Testing

```bash
# Dry run to see generated manifests
helm install vprofile ./vprofile --dry-run --debug

# Template rendering
helm template vprofile ./vprofile

# Validate without installing
helm lint ./vprofile
```

## Access the Application

After installation, access the application via:

- `http://vprofile.posafoods.online` (main app)
- `http://hima.posafoods.online` (main app)
- `http://<domain>/naveen` (nginx - if enabled)

## Troubleshooting

```bash
# Check pod status
kubectl get pods

# View pod logs
kubectl logs -f <pod-name>

# Describe pod issues
kubectl describe pod <pod-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'

# Helm release status
helm status vprofile
```

## Notes

- Database password and RabbitMQ password are base64 encoded in the secret
- PVC uses dynamic provisioning with the specified storage class
- Init containers ensure dependencies are ready before app starts
- Ingress uses regex and path rewriting for /naveen route
