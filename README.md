# Device Metrics Exporter

Device Metrics Exporter exports metrics from AMD GPUs to collectors like Prometheus.

## Supported Platforms
  - Ubuntu 22.04

## ROCm version
  - ROCm 6.2

## Installation (Option 1): Helm Chart Deployment for Kubernetes
### Prerequisites
  - Kubernetes cluster is up and running
  - Helm tool is installed on the node with kubectl + kube config file to get access to the cluster
### Installation
1. Prepare ```values.yaml``` to setup the deployment parameters, for example:
    ```yaml
    platform: k8s
    nodeSelector: {} # add customized nodeSelector for metrics exporter daemonset
    image:
      repository: docker.io/rocm/device-metrics-exporter
      tag: v1.0.0
      pullPolicy: Always
      pullSecrets: "" # put name of docker-registry secret here if needed for exporter image
    service:
      type: NodePort # select NodePort or ClusterIP as metrics exporter's service type
      ClusterIP:
        port: 5000 # cluster internal service port
      NodePort:      
        port: 5000 # cluster internal service port
        nodePort: 32500 # external node port 
    configMap: "" # put name of configmap here if needed for customizing exported stats
    ```
  - (Optional) if you want to customize the exported stats, please create a configmap by using ```example/configmap.yaml``` (please modify the namespace to align with helm install command), and put the configmap name into ```values.yaml```.
  
2. Run `helm install` command to deploy exporter in your Kubernetes cluster: 

    ```helm install exporter https://github.com/ROCm/device-metrics-exporter/releases/download/v1.0.0/device-metrics-exporter-charts-v1.0.0.tgz -n mynamespace -f values.yaml```
### Update Config:
  - Option 1: you can directly modify the Kubernetes resource to modify the config, including modifying configmap, service, rbac or daemonset resources.
  - Option 2: you can prepare the updated `values.yaml` and do a helm chart upgrade: 
  
     ```helm upgrade exporter -n mynamespace -f updated_values.yaml```

### Uninstallation
  - Uninstall the helm charts by running: 

    ```helm uninstall exporter -n mynamespace```

## Installation (Option 2): Run as Standalone Docker Container
### Prerequisites
  - Ubuntu 22.04/24.04
  - ROCm 6.2
  - Docker (podman or another alternative can be used, but commands provided are for Docker)

### Installation
1. Clone rocm/device-metrics-exporter repo and cd into folder
    ```
    git clone https://github.com/rocm/device-metrics-exporter
    cd device-metrics-exporter
    ```

2. Run metrics-exporter standalone docker container
   ```
   docker run -itd --device=/dev/dri --device=/dev/kfd -v ./config:/etc/metrics  -p 5000:5000 --name exporter rocm/device-metrics-exporter:v1.0.0
   ```

3. Update prometheus.yml file to replace localhost with host.docker.internal
   ```
   sed -i 's/localhost:5000/host.docker.internal:5000/g' example/prometheus.yml
   ```

## Local Testing with Prometheus and Grafana
### Running Prometheus via Docker
  - Run Prometheus as standalone docker container
    ```
    docker run -itd -p 9090:9090 -v ./example/prometheus.yml:/etc/prometheus/prometheus.yml -v prometheus-data:/prometheus --add-host host.docker.internal:host-gateway --name prometheus prom/prometheus
    ```
  - Prometheus should now be accessable on http://localhost:9090
  - Check the Status > Target health page to ensure the metrics exporter target is Up

### Running Grafana via Docker
- Run Grafana as standalone docker container
   ```
   docker run --rm -d --name grafana -p 3000:3000 --add-host host.docker.internal:host-gateway grafana/grafana:latest 
   ```
- Grafana should now be accessable on http://localhost:3000. Username and password to login is `admin/admin`

Note: Be sure to use http://host.docker.internal:9090 when adding Prometheus as a Data connection in the Grafana UI  

### Install and run Grafana as a service (Alternative)
- Run the below commands to install Grafana on your system. See [Grafana docs](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/) for more details
    ```
    sudo apt-get install -y apt-transport-https software-properties-common wget
    
    sudo mkdir -p /etc/apt/keyrings/
    
    wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
    
    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt sources.list.d/grafana.list
    
    sudo apt-get update
    
    sudo apt-get install grafana

    ```
- Run the Grafana server
    ```
    sudo systemctl daemon-reload
    sudo systemctl start grafana-server
    sudo systemctl status grafana-server
    ```
