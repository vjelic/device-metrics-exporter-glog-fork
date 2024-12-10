# Kubernetes (Helm)

For Kubernetes environments, we provide a Helm chart for easy deployment.

- Prepare a `values.yaml` file:

```yaml
platform: k8s
nodeSelector: {} # Optional: Add custom nodeSelector
image:
  repository: docker.io/rocm/device-metrics-exporter
  tag: v1.0.0
  pullPolicy: Always
service:
  type: ClusterIP  # or NodePort
  ClusterIP:
    port: 5000
```

- Install using Helm:

```bash
helm install exporter \
  https://github.com/ROCm/device-metrics-exporter/releases/download/v1.0.0/device-metrics-exporter-charts-v1.0.0.tgz \
  -n mynamespace -f values.yaml --create-namespace
```

## Configuration

### Default Settings

- Metrics endpoint: `http://localhost:5000/metrics`
- Default configuration file: `/etc/metrics/config.json`