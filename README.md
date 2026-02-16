# ArgoCD GitOps Repository - Web API Test

This repository contains the GitOps configuration for deploying a test web API application using ArgoCD.

## Repository Structure

```
.
├── base/                      # Base Kubernetes manifests
│   ├── deployment.yaml        # Deployment configuration
│   ├── service.yaml           # Service configuration
│   └── kustomization.yaml     # Kustomize configuration
└── argocd/                    # ArgoCD application definitions
    └── application.yaml       # ArgoCD Application manifest
```

## Prerequisites

- Local Kubernetes cluster (e.g., minikube, kind, Docker Desktop)
- ArgoCD installed and running in the cluster
- Local Docker registry accessible at `localhost:5000`
- Web API image published to the local registry as `localhost:5000/argo-testing:latest`

## Setup Instructions

### 1. Configure the Repository URL

Edit `argocd/application.yaml` and update the `repoURL` field with your actual Git repository URL:

```yaml
source:
  repoURL: https://github.com/YOUR_USERNAME/argotesting.gitops.git
```

### 2. Deploy the ArgoCD Application

Apply the ArgoCD application manifest to your cluster:

```bash
kubectl apply -f argocd/application.yaml
```

### 3. Verify Deployment

Check the ArgoCD UI or use the CLI to verify the application status:

```bash
# Using ArgoCD CLI
argocd app get argo-testing

# Using kubectl
kubectl get applications -n argocd
```

### 4. Access the Application

Once deployed, the service will be available within the cluster:

```bash
# Port-forward to access locally
kubectl port-forward svc/argo-testing 8080:80

# Access the API
curl http://localhost:8080
```

## Configuration Details

### Docker Registry Access

The deployment is configured to pull images from a local registry (`localhost:5000`). For local Kubernetes clusters:

- **Docker Desktop**: The local registry should be accessible directly
- **minikube**: You may need to configure insecure registries
- **kind**: You may need to add registry configuration to the cluster

### Auto-Sync Policy

The application is configured with automatic sync enabled:
- **prune**: Automatically delete resources that are no longer defined
- **selfHeal**: Automatically sync when the cluster state deviates from Git
- **retry**: Retry failed sync operations with exponential backoff

### Resource Configuration

Current configuration:
- **Replicas**: 2
- **Memory Request**: 128Mi
- **Memory Limit**: 256Mi
- **CPU Request**: 100m
- **CPU Limit**: 500m
- **Health Checks**: Configured for `/health` endpoint

## Customization

### Update Image Tag

To deploy a specific version, update the image tag in `base/deployment.yaml`:

```yaml
image: localhost:5000/argo-testing:v1.0.0
```

### Scale Replicas

Modify the replica count in `base/deployment.yaml`:

```yaml
spec:
  replicas: 3
```

### Change Namespace

Update the target namespace in `argocd/application.yaml`:

```yaml
destination:
  namespace: your-namespace
```

## Troubleshooting

### Image Pull Errors

If you encounter image pull errors, ensure:
1. The local registry is running and accessible
2. The image exists in the registry: `docker images | grep argo-testing`
3. Kubernetes is configured to trust the insecure registry

### ArgoCD Not Syncing

Check the application status:
```bash
argocd app get argo-testing
kubectl describe application argo-testing -n argocd
```

Common issues:
- Incorrect repository URL
- Repository not accessible from the cluster
- Invalid Kubernetes manifests

## Local Registry Setup (Reference)

If you need to set up a local registry:

```bash
# Run a local registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag and push your image
docker tag argo-testing localhost:5000/argo-testing:latest
docker push localhost:5000/argo-testing:latest
```
