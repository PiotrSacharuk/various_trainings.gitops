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