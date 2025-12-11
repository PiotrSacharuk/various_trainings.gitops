# 1. Check current status
kubectl get rollouts -w

# 2. GREEN is ready (no traffic yet)
kubectl describe rollout orderflow-orderflow

# 3. Test GREEN (via preview service)
curl http://orderflow-preview:5000/health

# 4. Promote GREEN → BLUE
kubectl argo rollouts promote orderflow-orderflow

# 5. Traffic switches (instant!)
# Rollback available:
kubectl argo rollouts abort orderflow-orderflow
