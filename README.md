# Multi-Environment GitOps Pipeline

A comprehensive DevOps project demonstrating modern GitOps practices with Kubernetes, Helm, and ArgoCD for managing multi-environment deployments.

## 🏗️ Architecture Overview

This project implements a GitOps-based CI/CD pipeline that manages application deployments across multiple environments (development, staging, and production) using declarative configuration and automated synchronization.

<img width="7156" height="3353" alt="Multi-Environment GitOps Pipeline" src="https://github.com/user-attachments/assets/d42e792d-a162-42bd-b064-f278d1898d49" />

### Key Components

- **Helm Charts**: Templated Kubernetes manifests for consistent deployments
- **ArgoCD Applications**: GitOps continuous delivery tool for automated deployments
- **Multi-Environment Configuration**: Environment-specific value overrides
- **Automated Sync**: Self-healing deployments with automated pruning

## 📁 Project Structure

```
├── README.md
├── applications/           # ArgoCD Application definitions
│   ├── dev-app.yaml       # Development environment app
│   ├── prod-app.yaml      # Production environment app
│   └── staging-app.yaml   # Staging environment app
├── environments/          # Environment-specific configurations
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── values.yaml    # Development overrides
│   ├── prod/
│   │   ├── kustomization.yaml
│   │   └── values.yaml    # Production overrides
│   └── staging/
│       ├── kustomization.yaml
│       └── values.yaml    # Staging overrides
└── helm/                  # Helm chart definition
    └── my-app/
        ├── Chart.yaml     # Chart metadata
        ├── values.yaml    # Default values
        └── templates/     # Kubernetes manifests
            ├── _helpers.tpl
            ├── deployment.yaml
            ├── ingress.yaml
            └── service.yaml
```

## 🚀 Features

### GitOps Workflow

- **Declarative Configuration**: All infrastructure and application configs stored in Git
- **Automated Deployment**: ArgoCD monitors Git repository and automatically deploys changes
- **Self-Healing**: Applications automatically recover from configuration drift
- **Rollback Capability**: Easy rollback to previous versions using Git history

### Multi-Environment Support

- **Development**: Single replica, minimal resources, development settings
- **Staging**: 2 replicas, moderate resources, staging configuration
- **Production**: 6 replicas, high resources, production-grade settings

### Environment Configurations

| Environment | Replicas | CPU Limit | Memory Limit | Node Port | NODE_ENV    |
| ----------- | -------- | --------- | ------------ | --------- | ----------- |
| Development | 1        | 100m      | 128Mi        | 30080     | development |
| Staging     | 2        | 200m      | 256Mi        | 30081     | staging     |
| Production  | 6        | 500m      | 512Mi        | 30082     | production  |

## 🛠️ Technology Stack

- **Kubernetes**: Container orchestration platform
- **Helm**: Package manager for Kubernetes
- **ArgoCD**: GitOps continuous delivery tool
- **Docker**: Containerization platform
- **YAML**: Configuration and manifest files

## 📋 Prerequisites

- Kubernetes cluster (minikube, kind, or cloud provider)
- ArgoCD installed and configured
- Helm 3.x installed
- kubectl configured to access your cluster
- Docker registry access

## 🔧 Setup Instructions

### 1. Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Access ArgoCD UI

```bash
# Get ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### 3. Deploy Applications

```bash
# Deploy all environments
kubectl apply -f applications/dev-app.yaml
kubectl apply -f applications/staging-app.yaml
kubectl apply -f applications/prod-app.yaml
```

### 4. Verify Deployments

```bash
# Check ArgoCD applications
kubectl get applications -n argocd

# Check deployments in each namespace
kubectl get deployments -n dev
kubectl get deployments -n staging
kubectl get deployments -n prod
```

## 🔄 GitOps Workflow

1. **Code Changes**: Developer pushes changes to the repository
2. **ArgoCD Detection**: ArgoCD detects changes in the Git repository
3. **Automatic Sync**: ArgoCD automatically applies changes to the cluster
4. **Health Monitoring**: ArgoCD monitors application health and status
5. **Self-Healing**: Any configuration drift is automatically corrected

## 🎯 Application Features

### Health Checks

- **Liveness Probe**: HTTP GET `/health` endpoint
- **Readiness Probe**: HTTP GET `/health` endpoint
- **Graceful Startup**: 30-second initial delay for liveness

### Security & Resource Management

- **Resource Limits**: CPU and memory constraints per environment
- **Image Pull Policy**: Configurable pull strategy
- **Namespace Isolation**: Separate namespaces for each environment

### Service Configuration

- **NodePort Service**: External access via node ports
- **Environment-Specific Ports**: Different ports for each environment
- **Target Port**: Application runs on port 3000

## 📊 Monitoring & Observability

The application includes built-in health endpoints for monitoring:

- **Health Check Endpoint**: `/health`
- **Kubernetes Probes**: Automated health monitoring
- **ArgoCD Dashboard**: Real-time deployment status
- **Resource Metrics**: CPU and memory usage tracking

## 🔐 Security Considerations

- **Namespace Isolation**: Each environment runs in separate namespaces
- **Resource Quotas**: Limited CPU and memory per environment
- **Image Security**: Specific image tags (avoid `latest` in production)
- **Network Policies**: Consider implementing for production workloads

## 🚀 Scaling Strategy

### Horizontal Scaling

- **Development**: 1 replica (minimal footprint)
- **Staging**: 2 replicas (load testing capability)
- **Production**: 6 replicas (high availability)

### Vertical Scaling

Resources scale proportionally across environments:

- **CPU**: 100m → 200m → 500m
- **Memory**: 128Mi → 256Mi → 512Mi

## 🔧 Customization

### Adding New Environments

1. Create environment-specific values file:

```bash
mkdir environments/new-env
touch environments/new-env/values.yaml
```

2. Create ArgoCD application:

```bash
cp applications/dev-app.yaml applications/new-env-app.yaml
# Edit the file to point to new environment
```

### Modifying Application Configuration

1. Update [helm/my-app/values.yaml](helm/my-app/values.yaml) for default changes
2. Update environment-specific files in [environments/](environments/) for per-environment changes
3. Commit changes to Git - ArgoCD will automatically deploy

## 🐛 Troubleshooting

### Common Issues

1. **Application Not Syncing**

   ```bash
   # Check ArgoCD application status
   kubectl describe application my-app-dev -n argocd
   ```

2. **Pod Startup Issues**

   ```bash
   # Check pod logs
   kubectl logs -l app.kubernetes.io/name=my-app -n dev
   ```

3. **Service Access Issues**
   ```bash
   # Verify service endpoints
   kubectl get endpoints -n dev
   ```

## 📈 Performance Metrics

### Resource Utilization by Environment

| Metric         | Development | Staging | Production |
| -------------- | ----------- | ------- | ---------- |
| CPU Request    | 50m         | 100m    | 200m       |
| CPU Limit      | 100m        | 200m    | 500m       |
| Memory Request | 64Mi        | 128Mi   | 256Mi      |
| Memory Limit   | 128Mi       | 256Mi   | 512Mi      |
| Replicas       | 1           | 2       | 6          |

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes and test thoroughly
4. Submit a pull request with detailed description

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👨‍💻 Author

**Vatsal Gajjar**

- GitHub: [@gajjarvatsall](https://github.com/gajjarvatsall)
- Project: Multi-Environment GitOps Pipeline

## 🙏 Acknowledgments

- ArgoCD community for excellent GitOps tooling
- Helm community for Kubernetes package management
- Kubernetes community for container orchestration platform

---

_This project demonstrates modern DevOps practices including GitOps, Infrastructure as Code, and multi-environment deployment strategies._
