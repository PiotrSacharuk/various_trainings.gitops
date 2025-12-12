Additional REQ:
istio (service mesh)


Day 1: Deploy 0.2.0 (A/B split 50/50)

ArgoCD updates values.yaml
  ↓
Deployment A: 0.1.0 (stable)
Deployment B: 0.2.0 (test)
  ↓
Istio: 50% traffic → A, 50% → B
  ↓
Metrics start collecting...

Day 2-7: Monitoring

Grafana dashboard observes:

Version A (0.1.0):
  Error rate: 0.5%
  Latency p95: 150ms
  Conversions: 2.3%

Version B (0.2.0):
  Error rate: 0.2% ✅ (better!)
  Latency p95: 120ms ✅ (better!)
  Conversions: 2.5% ✅ (better!)

Day 8: Decision
Data shows: 0.2.0 better in all metrics!

Deploy 0.2.0 to 100%:

VirtualService:
  - destination: orderflow-b
    weight: 100  # ← Change

Deployment A: can be disabled
  (or keep for instant rollback)

Rollback (if bug in 0.2.0)
VirtualService:
  - destination: orderflow-a
    weight: 100  # ← Back to A

All users now see 0.1.0 (safe)