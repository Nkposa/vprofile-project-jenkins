# Quick Start Guide - Vprofile Helm Deployment

## Complete Step-by-Step Instructions

### Prerequisites Check

```bash
# Check Kubernetes cluster
kubectl cluster-info

# Check Helm version (need 3.0+)
helm version

# Check current context
kubectl config current-context
```

---

## Step 1: Install Nginx Ingress Controller

```bash
# Add Helm repo
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install ingress controller
helm install ingress-nginx ingress-nginx/ingress-nginx \
  -n ingress-nginx \
  --create-namespace

# Wait for LoadBalancer IP (on AWS/cloud)
kubectl get svc -n ingress-nginx -w
```

---

## Step 2: Install Vprofile Application

```bash
# Navigate to helm charts directory
cd ~/vprofile-project/helm-charts

# Validate chart
helm lint ./vprofile

# Dry run (optional - to see what will be created)
helm install vprofile ./vprofile --dry-run --debug

# Install the release
helm install vprofile ./vprofile

# Or install in a specific namespace
helm install vprofile ./vprofile -n vprofile --create-namespace
```

---

## Step 3: Verify Deployment

```bash
# Check Helm release
helm list

# Get release status
helm status vprofile

# Watch pods coming up
kubectl get pods -w

# Check all resources
kubectl get all

# Check ingress
kubectl get ingress

# Check services
kubectl get svc

# Check PVC
kubectl get pvc
```

---

## Step 4: Access Application

```bash
# Get LoadBalancer IP/hostname
kubectl get svc -n ingress-nginx

# Update DNS records to point to LoadBalancer
# vprofile.posafoods.online -> <LoadBalancer-IP>
# hima.posafoods.online -> <LoadBalancer-IP>

# Test locally (optional)
curl http://vprofile.posafoods.online
curl http://hima.posafoods.online/naveen
```

---

## Step 5: Customize Values (Optional)

### Create custom values file

```bash
cat > custom-values.yaml << 'EOF'
app:
  replicas: 3  # Scale to 3 replicas

db:
  storage:
    size: 5Gi  # Increase storage

nginx:
  enabled: false  # Disable nginx
EOF

# Upgrade with custom values
helm upgrade vprofile ./vprofile -f custom-values.yaml
```

---

## Step 6: Upgrade Application

```bash
# After making changes to templates or values.yaml
helm upgrade vprofile ./vprofile

# Upgrade and force recreation
helm upgrade vprofile ./vprofile --force

# View upgrade history
helm history vprofile
```

---

## Step 7: Rollback (if needed)

```bash
# Rollback to previous version
helm rollback vprofile

# Rollback to specific revision
helm rollback vprofile 1

# Check history
helm history vprofile
```

---

## Step 8: Cleanup / Uninstall

```bash
# Uninstall vprofile application
helm uninstall vprofile

# Delete namespace (if created)
kubectl delete namespace vprofile

# Uninstall ingress controller
helm uninstall ingress-nginx -n ingress-nginx

# Delete ingress namespace
kubectl delete namespace ingress-nginx

# Verify everything is deleted
kubectl get all
helm list
kubectl get pvc
```

---

## Common Commands Reference

| Task | Command |
|------|---------|
| **Install** | `helm install vprofile ./vprofile` |
| **List releases** | `helm list` |
| **Status** | `helm status vprofile` |
| **Upgrade** | `helm upgrade vprofile ./vprofile` |
| **Rollback** | `helm rollback vprofile` |
| **Uninstall** | `helm uninstall vprofile` |
| **History** | `helm history vprofile` |
| **Get values** | `helm get values vprofile` |
| **Dry-run** | `helm install vprofile ./vprofile --dry-run` |
| **Lint** | `helm lint ./vprofile` |

---

## Troubleshooting

### Pods not starting

```bash
# Check pod status
kubectl get pods

# Describe pod for details
kubectl describe pod <pod-name>

# View logs
kubectl logs -f <pod-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

### Database issues

```bash
# Check PVC status
kubectl get pvc

# Check if volume is bound
kubectl describe pvc db-pv-claim

# Check database pod logs
kubectl logs -f deployment/vprodb
```

### Ingress not working

```bash
# Check ingress status
kubectl get ingress

# Describe ingress
kubectl describe ingress vpro-ingress

# Check ingress controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

### Secret issues

```bash
# Check secret exists
kubectl get secret app-secret

# Describe secret (without revealing values)
kubectl describe secret app-secret
```

---

## Quick Cleanup Script

```bash
#!/bin/bash
# cleanup.sh - Complete cleanup

echo "Uninstalling vprofile..."
helm uninstall vprofile

echo "Uninstalling ingress controller..."
helm uninstall ingress-nginx -n ingress-nginx

echo "Deleting namespaces..."
kubectl delete namespace ingress-nginx --ignore-not-found

echo "Deleting PVCs..."
kubectl delete pvc --all

echo "Cleanup complete!"
kubectl get all
```

---

## Success Indicators

✅ All pods in Running state
✅ PVC in Bound state
✅ Ingress has an ADDRESS
✅ Services have ClusterIP assigned
✅ Application accessible via domain

---

## Next Steps

1. Configure DNS to point to LoadBalancer
2. Optionally add TLS/SSL certificates
3. Configure monitoring (Prometheus/Grafana)
4. Setup auto-scaling (HPA)
5. Configure backups for database

Happy Helming! 🚀
