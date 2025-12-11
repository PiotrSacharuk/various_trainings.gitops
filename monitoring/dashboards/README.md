# Grafana Dashboards

This directory contains custom Grafana dashboard definitions in JSON format.

## Available Dashboards

- **cpu-usage-pod.json** - CPU usage metrics per pod
- **error-rate.json** - Error rate percentage tracking
- **error-requests.json** - Detailed error request analysis
- **memory-usage-pod.json** - Memory usage metrics per pod
- **pod-status.json** - Pod status overview
- **requests-by-endpoint.json** - Request distribution by endpoint
- **requests-per-second.json** - RPS (Requests Per Second) monitoring

## Importing Dashboards

### Manual Import
1. Open Grafana UI (http://localhost:3000)
2. Navigate to **Dashboards → Import**
3. Upload the JSON file or paste its contents
4. Configure data source if prompted
5. Click **Import**

### Automated Import (GitOps)
To automatically provision these dashboards via ConfigMaps:

```bash
kubectl create configmap grafana-dashboard-<name> \
  --from-file=<dashboard-file>.json \
  -n monitoring \
  --dry-run=client -o yaml | \
  kubectl label -f - grafana_dashboard=1 --local -o yaml | \
  kubectl apply -f -
```

Example:
```bash
kubectl create configmap grafana-dashboard-cpu-usage \
  --from-file=cpu-usage-pod.json \
  -n monitoring \
  --dry-run=client -o yaml | \
  kubectl label -f - grafana_dashboard=1 --local -o yaml | \
  kubectl apply -f -
```

The Grafana sidecar will automatically detect ConfigMaps with the `grafana_dashboard=1` label and load them.
