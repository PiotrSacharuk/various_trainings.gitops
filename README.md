# various-trainings.gitops

nohup kubectl port-forward svc/argocd-server -n argocd 8080:443 &

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

helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  -f monitoring/prometheus-values.yaml \
  --set grafana.adminPassword=<set your password here>

# GRAFANA LOKI + Promtail
sudo sysctl -w fs.inotify.max_user_watches=524288
sudo sysctl -w fs.inotify.max_user_instances=8192
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm upgrade --install loki grafana/loki-stack \
  -n monitoring \
  --set loki.enabled=true \
  -f monitoring/promtail-values.yaml

# ACCESS

# Prometheus (http://localhost:9090)
nohup kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090 -n monitoring &

# Grafana (http://localhost:3000) - login: admin / <your-password>
nohup kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring &


# SEALED SECRETS
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo update

helm install sealed-secrets sealed-secrets/sealed-secrets \
  --namespace kube-system \
  --set commandArgs="{--update-status}"

kubectl rollout status deployment/sealed-secrets-sealed-secrets -n kube-system

wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/kubeseal-0.24.0-linux-amd64.tar.gz
tar xfz kubeseal-0.24.0-linux-amd64.tar.gz
sudo mv kubeseal /usr/local/bin/

kubectl get secret -n kube-system \
  -l sealedsecrets.bitnami.com/status=active \
  -o jsonpath='{.items[0].data.tls\.crt}' | base64 -d > sealing-key.pub

# mysql-secret-plain.yaml is filled following template with values
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: mysql
type: Opaque
stringData:
  root-password:
  username:
  password:


kubeseal -f mysql-secret-plain.yaml \
  -w mysql-secret-sealed.yaml \
  --scope namespace-wide \
  --controller-name=sealed-secrets \
  --controller-namespace=kube-system