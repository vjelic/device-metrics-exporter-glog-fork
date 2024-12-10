# Docker

## Prerequisites

- Ubuntu 22.04 or later
- ROCm 6.2.0
- Docker (or a Docker-compatible container runtime)

## Installation

The Metrics Exporter container is hosted on Docker Hub at [rocm/device-metrics-exporter](https://hub.docker.com/r/rocm/device-metrics-exporter). 

- Start the container:

```bash
docker run -d \
  --device=/dev/dri \
  --device=/dev/kfd \
  -p 5000:5000 \
  --name device-metrics-exporter \
  rocm/device-metrics-exporter:v1.0.0
```

- Confirm metrics are accessible:

```bash
curl http://localhost:5000/metrics
```

- Review the [Prometheus and Grafana Integration Guide](../integrations/prometheus-grafana.md)

## Custom metrics

To run the Device Metrics Exporter with a custom config mount the `/etc/metrics/config.json` file on the exporter container.

1. Create your own config file in directory `config/config.json`. Example file in the metrics exporter repo: [https://raw.githubusercontent.com/ROCm/device-metrics-exporter/refs/heads/main/example/config.json](https://raw.githubusercontent.com/ROCm/device-metrics-exporter/refs/heads/main/example/config.json)