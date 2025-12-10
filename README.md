# various-trainings.gitops

follow https://github.com/PiotrSacharuk/ArgoCD scripts for argocd setup

kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=*** \
  --docker-password=*** \
  --docker-email=***

kubectl create namespace argo-rollouts
kubectl apply -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml -n argo-rollouts


# MANIFEST
project: default
source:
  repoURL: https://github.com/PiotrSacharuk/various-trainings.gitops
  path: orderflow
  targetRevision: HEAD
destination:
  server: https://kubernetes.default.svc
  namespace: default
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true

# INGRESS
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true


# PROMETHEUS
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/prometheus \
  --namespace monitoring \
  --create-namespace

helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values - <<EOF
prometheus:
  prometheusSpec:
    retention: 7d
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 5Gi

grafana:
  adminPassword: "admin123"
  persistence:
    enabled: true
    size: 5Gi

alertmanager:
  enabled: true
EOF