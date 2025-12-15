# Various Trainings - GitOps

A comprehensive Kubernetes GitOps training repository showcasing advanced deployment patterns, ArgoCD integration, observability stack, and cloud-native best practices.

## 📋 Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Directory Descriptions](#directory-descriptions)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Core Components](#core-components)
- [Deployment Patterns](#deployment-patterns)
- [Tools & Technologies](#tools--technologies)
- [Setup Instructions](#setup-instructions)
- [Useful Commands](#useful-commands)
- [Contributing](#contributing)

## 🎯 Overview

This repository is designed as a **practical learning resource** for cloud-native development with a focus on:

- **GitOps Principles**: Declarative infrastructure management using Git as the single source of truth
- **ArgoCD**: Continuous deployment and application lifecycle management
- **Kubernetes**: Container orchestration and workload management
- **Observability**: Monitoring, logging, and alerting with Prometheus, Grafana, and Loki
- **Advanced Patterns**: Blue-green deployments, A/B testing, database statefulsets
- **Security**: Secret management with Sealed Secrets, certificate management with cert-manager

## 📁 Repository Structure

```
various_trainings.gitops/
├── argocd-applications/              # ArgoCD Application definitions for production
│   ├── monitoring-app.yaml           # Monitoring stack deployment
│   ├── orderflow-app.yaml            # Production orderflow service
│   ├── orderflow-app-dev.yaml        # Development environment
│   ├── orderflow-app-staging.yaml    # Staging environment
│   └── sqlite-app.yaml               # SQLite database application
├── archive-argocd-applications/      # Legacy/archived application definitions
│   └── mysql-app.yaml                # MySQL database application
├── orderflow/                        # Orderflow service (Helm chart)
│   ├── Chart.yaml                    # Helm chart metadata
│   ├── values.yaml                   # Default Helm values
│   ├── templates/                    # Kubernetes manifest templates (HPA, Ingress, Rollout, Service, ServiceMonitor)
│   └── overlays/                     # Kustomize overlays for different environments
├── monitoring/                       # Prometheus and Grafana configurations
│   ├── prometheus-values.yaml        # Prometheus Helm values
│   ├── prometheus-rules.yaml         # PrometheusRule definitions and alerts
│   ├── promtail-values.yaml          # Promtail log collector configuration
│   └── dashboards/                   # Grafana dashboard JSON files
├── monitoring-daemonset/             # DaemonSet monitoring deployments
│   ├── Chart.yaml                    # Helm chart metadata
│   ├── values.yaml                   # MySQL configuration values
│   └── templates/                    # MySQL manifests (DaemonSet, ServiceAccount)
├── cert-manager/                     # Certificate management configurations
│   ├── base-issuer.yaml              # Base issuer configuration
│   ├── ca-certificate.yaml           # CA certificate configuration
│   └── ca-issuer.yaml                # CA issuer configuration
├── mysql-statefulset/                # MySQL database StatefulSet (Helm)
│   ├── Chart.yaml                    # Helm chart metadata
│   ├── values.yaml                   # MySQL configuration values
│   └── templates/                    # MySQL manifests (StatefulSet, Service, SealedSecret)
├── sqlite-statefulset/               # SQLite database StatefulSet (Helm)
│   ├── Chart.yaml                    # Helm chart metadata
│   ├── values.yaml                   # MySQL configuration values
│   └── templates/                    # MySQL manifests (ConfigMap, Service, StatefulSet)
└── README.md                         # This file

NOT TESTED
├── blue-green/                       # Blue-green deployment pattern
│   ├── rollout.yaml                  # Argo Rollouts configuration
│   ├── service.yaml                  # Kubernetes Service definition
│   └── README.md                     # Blue-green deployment guide
├── a-b-testing/                      # A/B testing deployment strategy
│   ├── deployment.yaml               # Deployment configuration
│   ├── destination-rule.yaml         # DestinationRule configuration
│   ├── grafana-queries               # Grafana queries for A/B testing
│   ├── prometheus-scape-config.yaml  # Prometheus scrape configuration
│   ├── virtual-service.yaml          # VirtualService configuration
│   └── README.md                     # A/B testing deployment guide

```

## 📖 Directory Descriptions

### 🔹 `argocd-applications/`
Contains all ArgoCD Application definitions used for continuous deployment. These files define which Kubernetes manifests to deploy, from which Git repository, and how to sync them.

**Key Files:**
- `monitoring-app.yaml`        - Deploys DaemonSet monitoring stack
- `orderflow-app.yaml`         - Production orderflow application
- `orderflow-app-dev.yaml`     - Development environment with auto-sync
- `orderflow-app-staging.yaml` - Staging environment for testing
- `sqlite-app.yaml`            - SQLite database deployment

**Example Usage:**
```bash
kubectl apply -f argocd-applications/orderflow-app.yaml
```

### 🔹 `orderflow/`
Helm chart for the orderflow service application. This is a templated Kubernetes application that can be customized through values.

**Files:**
- `Chart.yaml` - Helm chart metadata (name, version, description)
- `values.yaml` - Default configuration values (replicas, image, ports, resources)
- `templates/` - Kubernetes manifest templates (Deployment, Service, ConfigMap, Ingress)
- `overlays/` - Kustomize overlays for environment-specific configurations

**Key Concepts:**
- Helm templating for reusable configurations
- Support for multiple environments via overlays
- Parameterized deployments through values

**Deploy:**
```bash
helm install orderflow ./orderflow -f orderflow/values.yaml
```

### 🔹 `monitoring/`
Complete observability stack configuration for Prometheus, Grafana, Loki, and Promtail.

**Files:**
- `prometheus-values.yaml` - Prometheus Helm chart customization
  - Metrics scrape configuration
  - Retention policies
  - Storage settings

- `prometheus-rules.yaml` - Alert rules and recording rules
  - Pod CPU/Memory alerts
  - Pod restart alerts
  - Custom business metrics

- `promtail-values.yaml` - Log collection configuration
  - Log scrape jobs
  - Label configuration
  - Loki endpoint settings

- `dashboards/` - Grafana dashboard definitions
  - Pre-built dashboards for cluster monitoring
  - Application-specific dashboards
  - Business metrics dashboards

**Deploy Monitoring Stack:**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace -f monitoring/prometheus-values.yaml
```

### 🔹 `mysql-statefulset/`
Helm chart for MySQL database deployment as a StatefulSet with persistent storage.

**Files:**
- `Chart.yaml` - Helm chart metadata for MySQL
- `values.yaml` - MySQL configuration
  - Image and version
  - Storage size (PersistentVolumeClaim)
  - Resource limits
  - Database credentials

- `templates/` - Kubernetes manifests
  - `StatefulSet` - MySQL pod with persistent storage
  - `Service` - Headless and ClusterIP services
  - `Secret` - Database credentials
  - `PersistentVolumeClaim` - Storage configuration
  - `ConfigMap` - my.cnf MySQL configuration

**Key Features:**
- Persistent storage for data durability
- Headless service for DNS resolution
- Ordered, stable pod identities
- Backup and recovery capabilities

**Deploy MySQL:**
```bash
helm install mysql ./mysql-statefulset -n mysql --create-namespace
```

### 🔹 `sqlite-statefulset/`
Helm chart for SQLite database deployment. Lightweight alternative to MySQL for development.

**Files:**
- `Chart.yaml` - Helm chart metadata
- `values.yaml` - SQLite configuration
- `templates/` - Kubernetes manifests for SQLite deployment

**Use Case:**
- Development and testing environments
- Light workloads
- Single-node deployments

### 🔹 `blue-green/`
Implementation of blue-green deployment pattern using Argo Rollouts for zero-downtime deployments.

**Files:**
- `rollout.yaml` - Argo Rollouts Rollout resource
  - Blue and green ReplicaSet definitions
  - Traffic switching strategy
  - Analysis templates for automated validation
  - Canary or immediate traffic switch options

- `service.yaml` - Kubernetes Service
  - Label selectors for blue/green traffic routing
  - LoadBalancer or NodePort configuration
  - Health check probes

- `README.md` - Detailed blue-green deployment guide
  - Deployment strategy explanation
  - Traffic switching procedures
  - Rollback mechanisms

**Blue-Green Workflow:**
1. Deploy green version alongside blue
2. Run health checks on green
3. Switch all traffic to green
4. Keep blue as instant rollback point
5. Promote green to blue for next deployment

**Deploy Blue-Green:**
```bash
kubectl apply -f blue-green/rollout.yaml
kubectl apply -f blue-green/service.yaml
```

### 🔹 `a-b-testing/`
A/B testing deployment strategy for feature validation with real users.

**Files:**
- A/B testing configuration files
- Traffic splitting rules
- Metrics collection templates
- Success criteria definitions

**Use Case:**
- Test new features with subset of users
- Gradual rollout with traffic split (90/10, 75/25, etc.)
- Collect metrics to determine success
- Automated promotion based on metrics

### 🔹 `cert-manager/`
TLS certificate management using cert-manager for automatic certificate provisioning.

**Files:**
- Certificate definitions (Certificate CR)
- ClusterIssuer/Issuer configurations
- Let's Encrypt integration
- Self-signed certificate templates

**Features:**
- Automatic certificate generation
- Renewal before expiry
- DNS01 and HTTP01 validation
- Multi-environment support

### 🔹 `monitoring-daemonset/`
DaemonSet-based monitoring deployments that run on every node.

**Purpose:**
- Node-level metrics collection
- Per-node log collection
- Network monitoring
- Security monitoring

### 🔹 `archive-argocd-applications/`
Legacy and archived ArgoCD application definitions for reference.

**Contents:**
- Deprecated deployment configurations
- Historical examples
- Alternative deployment strategies
- Migration guides

---

## 🔧 Prerequisites

- **Kubernetes Cluster** (v1.24+)
  - Recommended: k3d for local development
  - TODO:Or: Any managed Kubernetes service (EKS, AKS, GKE)
- **kubectl** (v1.24+)
- **Helm** (v3.0+)
- **ArgoCD CLI** (optional but recommended)
- **kubeseal** (for Sealed Secrets management)
- **Git** (for GitOps workflows)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/PiotrSacharuk/various_trainings.gitops.git
cd various_trainings.gitops
```

### 2. Install ArgoCD

```bash
# Create argocd namespace
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl rollout status deployment/argocd-server -n argocd
```

### 3. Access ArgoCD UI

```bash
# Port forward to ArgoCD server
nohup kubectl port-forward svc/argocd-server -n argocd 8080:443 &

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Access UI at https://localhost:8080
# Login with: admin / <password-from-above>
```

### 4. Create Docker Registry Secret (if using private registries)

```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=YOUR_USERNAME \
  --docker-password=YOUR_PAT \
  --docker-email=YOUR_EMAIL
```

### 5. Deploy Your First Application

```bash
# Deploy orderflow application
kubectl apply -f argocd-applications/orderflow-app.yaml

# Or deploy monitoring stack
kubectl apply -f argocd-applications/monitoring-app.yaml
```

## 🏗️ Core Components

### ArgoCD Applications

The repository includes several ArgoCD Application manifests for different environments:

| Application | Purpose | Path | Sync Policy |
|---|---|---|---|
| **Orderflow (Production)** | Main production application | `argocd-applications/orderflow-app.yaml` | Manual + Self-Healing |
| **Orderflow (Development)** | Development environment | `argocd-applications/orderflow-app-dev.yaml` | Auto-sync |
| **Orderflow (Staging)** | Staging environment | `argocd-applications/orderflow-app-staging.yaml` | Manual |
| **Monitoring Stack** | Prometheus & Grafana | `argocd-applications/monitoring-app.yaml` | Auto-sync |
| **SQLite Database** | SQLite StatefulSet | `argocd-applications/sqlite-app.yaml` | Manual + Self-Healing |

### Database Solutions

#### MySQL StatefulSet
- **Path**: `mysql-statefulset/`
- **Type**: Helm Chart
- **Storage**: PersistentVolumeClaim (configurable size)
- **Replicas**: Single primary (scalable to multi-replica)
- **Use Case**: Production databases, complex queries, transactions

**Key Files:**
- `values.yaml` - Storage size, image version, credentials
- `templates/StatefulSet.yaml` - Pod lifecycle management
- `templates/Service.yaml` - DNS and network access

#### SQLite StatefulSet
- **Path**: `sqlite-statefulset/`
- **Type**: Helm Chart
- **Storage**: Embedded file-based database
- **Use Case**: Development, testing, lightweight deployments
- **Advantages**: No separate database server, simple backups

### Observability Stack

#### Prometheus
- **Path**: `monitoring/prometheus-values.yaml`
- **Metrics**: Collects time-series metrics from all pods
- **Scrape Targets**: Kubelet, API server, custom applications
- **Storage**: Time-series database with retention policies
- **Access**: http://localhost:9090 (after port-forward)

#### Grafana
- **Purpose**: Visualize Prometheus metrics
- **Dashboards**: Pre-built cluster and application dashboards
- **Access**: http://localhost:3000 (after port-forward)
- **Default Credentials**: admin / YOUR_PASSWORD

#### Loki
- **Purpose**: Log aggregation (lightweight Prometheus-like for logs)
- **Promtail**: Collects logs from containers
- **Configuration**: `monitoring/promtail-values.yaml`
- **Query Language**: LogQL (similar to PromQL)

#### Promtail
- **Purpose**: Log shipping agent
- **Configuration**: `monitoring/promtail-values.yaml`
- **Deployment**: DaemonSet (one per node)
- **Targets**: Pod logs, system logs, application logs

### Security

- **Sealed Secrets**: Encrypted secret management in Git
- **cert-manager**: Automatic TLS certificate provisioning
- **TLS/SSL**: Secure ingress with Let's Encrypt
- **RBAC**: Role-based access control for cluster access

## 🔄 Deployment Patterns

### Blue-Green Deployment

Separate but identical production environments. Switch traffic between them for zero-downtime deployments.

**Location**: `blue-green/`

**Files:**
- `rollout.yaml` - Argo Rollouts Rollout resource with blue-green strategy
- `service.yaml` - Kubernetes Service for traffic switching
- `README.md` - Detailed implementation guide

**Workflow:**
```
1. Current state (Blue): v1 (active, receives all traffic)
   ↓
2. Deploy (Green): v2 (in parallel)
   ↓
3. Validate Green: Run health checks and tests
   ↓
4. Switch Traffic: Route all traffic to Green (v2)
   ↓
5. Promote: Green becomes Blue (v2 is now active)
   ↓
6. Keep Blue: v1 remains as rollback point
```

**Advantages:**
- Zero-downtime deployments
- Instant rollback capability
- Easy troubleshooting on v1
- A/B testing capability

### A/B Testing

Run multiple application versions simultaneously to test new features with subset of users.

**Location**: `a-b-testing/`

**Strategy:**
- Route 90% traffic to stable version
- Route 10% traffic to new version
- Collect metrics from both
- Gradually increase traffic if metrics are good
- Rollback if issues detected

**Use Cases:**
- Feature flag testing
- UI/UX experiments
- Performance comparison
- Gradual feature rollout

## 🛠️ Tools & Technologies

| Component | Version | Purpose | Location |
|-----------|---------|---------|----------|
| **Kubernetes** | 1.24+ | Container orchestration | All |
| **ArgoCD** | Latest | Continuous deployment | `argocd-applications/` |
| **Prometheus** | Latest | Metrics collection | `monitoring/prometheus-*` |
| **Grafana** | Latest | Metrics visualization | `monitoring/` |
| **Loki** | Latest | Log aggregation | `monitoring/promtail-*` |
| **cert-manager** | 1.x | Certificate management | `cert-manager/` |
| **Sealed Secrets** | 0.24+ | Secret encryption | Referenced in guides |
| **Argo Rollouts** | Latest | Advanced deployments | `blue-green/`, `a-b-testing/` |
| **Helm** | 3.0+ | Package management | All Helm charts |
| **Kustomize** | Latest | Config templating | `orderflow/overlays/` |

## 📚 Setup Instructions

### Install Argo Rollouts

```bash
kubectl create namespace argo-rollouts
kubectl apply -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml -n argo-rollouts
```

### Deploy MySQL StatefulSet

```bash
# Using Helm directly
helm install mysql ./mysql-statefulset \
  --namespace mysql \
  --create-namespace \
  --set storage.size=10Gi

# Using ArgoCD (if configured)
kubectl apply -f argocd-applications/mysql-app.yaml
```

### Deploy SQLite Application

```bash
helm install sqlite ./sqlite-statefulset \
  --namespace sqlite \
  --create-namespace
```

### Install and Configure Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  -f monitoring/prometheus-values.yaml \
  --set grafana.adminPassword=YOUR_SECURE_PASSWORD

# Verify installation
kubectl rollout status deployment/prometheus-operator -n monitoring
```

### Install Grafana with Loki

```bash
# Increase inotify limits (required for log collection)
sudo sysctl -w fs.inotify.max_user_watches=524288
sudo sysctl -w fs.inotify.max_user_instances=8192

# Add Grafana Helm repository
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Loki stack with Promtail
helm upgrade --install loki grafana/loki-stack \
  -n monitoring \
  --create-namespace \
  --set loki.enabled=true \
  -f monitoring/promtail-values.yaml \
  --set promtail.enabled=true

# Verify installation
kubectl get pods -n monitoring | grep loki
```

### Install cert-manager

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true

# Verify installation
kubectl rollout status deployment/cert-manager -n cert-manager
```

### Install Sealed Secrets

```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo update

helm install sealed-secrets sealed-secrets/sealed-secrets \
  --namespace kube-system \
  --set commandArgs="{--update-status}"

# Verify installation
kubectl rollout status deployment/sealed-secrets-sealed-secrets -n kube-system

# Download kubeseal CLI tool
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/kubeseal-0.24.0-linux-amd64.tar.gz
tar xfz kubeseal-0.24.0-linux-amd64.tar.gz
sudo mv kubeseal /usr/local/bin/

# Export public sealing key for reference
kubectl get secret -n kube-system \
  -l sealedsecrets.bitnami.com/status=active \
  -o jsonpath='{.items[0].data.tls\.crt}' | base64 -d > sealing-key.pub
```

## 📝 Useful Commands

### ArgoCD Management

```bash
# Port forward to ArgoCD server
nohup kubectl port-forward svc/argocd-server -n argocd 8080:443 &

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# List all ArgoCD applications
kubectl get applications -n argocd

# Get detailed application status
argocd app get orderflow

# Manually sync an application
argocd app sync orderflow

# Watch application sync progress
kubectl logs -f deployment/argocd-application-controller -n argocd
```

### Helm Operations

```bash
# List installed releases
helm list --all-namespaces

# Upgrade a release with new values
helm upgrade mysql ./mysql-statefulset \
  --namespace mysql \
  --set storage.size=20Gi

# Rollback to previous release
helm rollback mysql --namespace mysql

# Get release values
helm get values mysql --namespace mysql

# Template rendering (dry-run)
helm template orderflow ./orderflow -f orderflow/values.yaml
```

### Sealed Secrets

```bash
# Create a plain secret file
cat > mysql-secret-plain.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: mysql
type: Opaque
stringData:
  root-password: your-secure-password
  user: root
  password: your-app-password
EOF

# Seal the secret
kubeseal -f mysql-secret-plain.yaml \
  -w mysql-secret-sealed.yaml \
  --scope namespace-wide \
  --controller-name=sealed-secrets \
  --controller-namespace=kube-system

# Apply sealed secret (safe to commit to Git)
kubectl apply -f mysql-secret-sealed.yaml

# Verify sealed secret
kubectl get sealedsecrets -n mysql
```

### Monitoring & Observability

```bash
# Port forward Prometheus
nohup kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090 -n monitoring &

# Port forward Grafana
nohup kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring &

# Port forward Loki (optional)
nohup kubectl port-forward svc/loki 3100:3100 -n monitoring &

# Query Prometheus from CLI
kubectl exec -it <prometheus-pod> -n monitoring -- promtool query instant 'up'

# View pod logs
kubectl logs -f pod/orderflow-abc123 -n default

# Stream logs with Loki (using logcli)
logcli query '{job="kubernetes"}'
```

### Database Management

```bash
# Connect to MySQL Pod
kubectl exec -it mysql-0 -n mysql -- mysql -u root -p

# View MySQL StatefulSet
kubectl get statefulsets -n mysql
kubectl describe statefulset mysql -n mysql

# Check persistent volumes
kubectl get pv
kubectl get pvc --all-namespaces

# Monitor PVC usage
kubectl exec -it mysql-0 -n mysql -- df -h /var/lib/mysql

# Backup MySQL
kubectl exec -it mysql-0 -n mysql -- mysqldump -u root -p --all-databases > backup.sql

# Restore from backup
kubectl exec -it mysql-0 -n mysql -- mysql -u root -p < backup.sql
```

### Deployment Pattern Monitoring

```bash
# Watch blue-green deployment
kubectl get rollouts.argoproj.io -n default -w

# Check rollout status
kubectl argo rollouts status orderflow -n default

# Promote rollout to next step
kubectl argo rollouts promote orderflow -n default

# Abort rollout and rollback
kubectl argo rollouts abort orderflow -n default
```

## 📖 Example ArgoCD Application Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: orderflow
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/PiotrSacharuk/various_trainings.gitops
    path: orderflow
    targetRevision: HEAD
    helm:
      releaseName: orderflow
      values: |
        replicaCount: 3
        image:
          repository: ghcr.io/yourusername/orderflow
          tag: "1.0.0"
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

## 🔐 Security Best Practices

1. **Use Sealed Secrets** for sensitive data instead of plain Kubernetes secrets
2. **Enable RBAC** with proper role definitions and least privilege access
3. **Use network policies** to restrict traffic between pods
4. **Scan images** for vulnerabilities before deployment with tools like Trivy
5. **Implement admission controllers** for policy enforcement (Pod Security, Network Policy)
6. **Use TLS** for all ingress endpoints with cert-manager and Let's Encrypt
7. **Rotate credentials** regularly and use strong passwords (24+ characters)
8. **Audit logs** for compliance and troubleshooting using Kubernetes audit logs
9. **Use private container registries** with authentication
10. **Implement resource quotas** to prevent resource exhaustion
11. **Use service accounts** with minimal required permissions
12. **Enable encryption at rest** for persistent volumes (if supported by infrastructure)

## 📚 Learning Resources

- 📖 [ArgoCD Documentation](https://argo-cd.readthedocs.io/) - GitOps deployment automation
- 📖 [Kubernetes Documentation](https://kubernetes.io/docs/) - Container orchestration
- 📖 [Helm Documentation](https://helm.sh/docs/) - Kubernetes package manager
- 📖 [Prometheus Documentation](https://prometheus.io/docs/) - Metrics monitoring
- 📖 [Grafana Documentation](https://grafana.com/docs/) - Metrics visualization
- 📖 [Argo Rollouts](https://argoproj.github.io/argo-rollouts/) - Advanced deployment strategies
- 📖 [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) - Encrypted secrets in Git
- 📖 [cert-manager](https://cert-manager.io/docs/) - Automatic certificate management

## 🤝 Contributing

Contributions are welcome! Please feel free to:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Make your changes and test them locally
4. Commit your changes (`git commit -am 'Add improvement'`)
5. Push to the branch (`git push origin feature/improvement`)
6. Open a Pull Request with a description of your changes

## 📄 License

This project is provided as-is for educational and training purposes.

## 👤 Author

Created and maintained by [Piotr Sacharuk](https://github.com/PiotrSacharuk)

## 💬 Support & Issues

For questions, issues, or suggestions:
1. Check existing [GitHub Issues](https://github.com/PiotrSacharuk/various_trainings.gitops/issues)
2. Review the directory-specific README files
3. Open a new issue with:
   - Clear description of the problem
   - Steps to reproduce
   - Expected vs actual behavior
   - Kubernetes version and cluster type
   - Relevant logs or error messages

---

**Last Updated**: December 2025

**Status**: Active - Continuously used to train on latest GitOps and Kubernetes best practices

**Tested On:**
- Kubernetes 1.24+
- k3d local clusters
