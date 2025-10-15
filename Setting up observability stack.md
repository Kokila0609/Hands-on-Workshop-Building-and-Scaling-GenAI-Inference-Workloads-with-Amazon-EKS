**Setting up observability stack**

In this section, we'll explore the core components of our observability stack designed to monitor LLM inference workloads on Amazon EKS.

**Observability Stack Components**

For monitoring LLM inference workloads, we have deployed several key components in the monitoring namespace that form our observability stack:

**Kube Prometheus Stack**

The Kube Prometheus Stack provides a complete monitoring solution. Let's examine each component:

``` bash
# View all components in the monitoring namespace
kubectl get pods -n monitoring

# Check Prometheus deployment
kubectl get pods -l "app.kubernetes.io/name=prometheus" -n monitoring

# Check Node Exporter DaemonSet
kubectl get pods -l "app.kubernetes.io/name=prometheus-node-exporter" -n monitoring

# Check Kube State Metrics deployment
kubectl get pods -l "app.kubernetes.io/name=kube-state-metrics" -n monitoring

Each component serves a specific purpose:

Prometheus Server: Central metrics collection and storage

Node Exporter: Collects hardware and OS metrics from each node (runs as DaemonSet)

Kube State Metrics: Generates metrics about Kubernetes objects

Grafana Stack
Our Grafana setup includes both the Grafana server and Grafana Operator to provision Dashboards using YAML files:

1
2
3
4
5
# Check Grafana Server deployment
kubectl get pods -l "app.kubernetes.io/name=grafana" -n monitoring

# Check Grafana Operator deployment
kubectl get pods -l "app.kubernetes.io/name=grafana-operator" -n monitoring

Grafana Operator
Grafana Operator is being used to create Grafana dashboards using custom resources. Use the following command to check the configuration:

1
kubectl get Grafana external-grafana -n monitoring -o yaml
```

Installing NVIDIA DCGM Exporter
The NVIDIA Data Center GPU Manager (DCGM) Exporter  is a tool that exposes GPU metrics for NVIDIA GPUs. It collects essential metrics such as GPU utilization, memory usage, temperature, power consumption, and other performance indicators. These metrics are exported in a Prometheus format, making it ideal for monitoring GPU workloads in Kubernetes environments.

Let's create a values file for the DCGM Exporter configuration:

``` bash
# Create values.yaml file for DCGM Exporter
mkdir -p manifests/200-inference
cat << EOF > manifests/200-inference/values.yaml
serviceMonitor:
  enabled: true
  namespace: monitoring
  additionalLabels:
    release: kube-prometheus-stack  # Important for prometheus operator discovery
  interval: 30s
  scrapeTimeout: 10s
  honorLabels: true

service:
  type: ClusterIP
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/app-metrics: "true"
    prometheus.io/port: "9400"
  labels:
    app.kubernetes.io/name: "dcgm-exporter"

nodeSelector:
  "nvidia.com/gpu.present": "true"

tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"

pod:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9400"
  labels:
    app.kubernetes.io/name: "dcgm-exporter"

dcgmExporter:
  env:
    - name: "DCGM_EXPORTER_LISTEN"
      value: ":9400"
    - name: "DCGM_EXPORTER_KUBERNETES"
      value: "true"
EOF
```

We'll now install the NVIDIA DCGM Exporter to collect GPU metrics:

``` bash
# Add NVIDIA Helm repository
helm repo add gpu-helm-charts https://nvidia.github.io/dcgm-exporter/helm-charts
helm repo update

# Install DCGM Exporter in the monitoring namespace
helm install dcgm-exporter gpu-helm-charts/dcgm-exporter \
  -n monitoring \
  -f manifests/200-inference/values.yaml

# Verify DCGM Exporter deployment
kubectl wait pods --for=jsonpath='{.status.phase}'=Running -l "app.kubernetes.io/name=dcgm-exporter" -n monitoring --timeout=300s

```


**Verifying GPU Metrics**

To verify that DCGM Exporter is collecting GPU metrics correctly, open a new terminal window and follow the instructions below:

In another terminal, setup the port-forward:

```
# Get the name of a DCGM Exporter pod
NAME=$(kubectl get pods -l "app.kubernetes.io/name=dcgm-exporter" \
                       -n monitoring \
                       -o "jsonpath={ .items[0].metadata.name}")

# Set up port forwarding to access the metrics endpoint
kubectl port-forward -n monitoring $NAME 8080:9400

Check /metrics endpoint of the DCGM exporter pods to view GPU metrics.


# Query the metrics endpoint
curl -sL http://127.0.0.1:8080/metrics
```

**Conclusion**

In this section, we have:

✅ Verified our existing monitoring stack components:

Kube Prometheus Stack
Grafana and Grafana Operator
Alert Manager
Node Exporter
Kube State Metrics

✅ Successfully installed NVIDIA DCGM Exporter:

Configured it to run only on GPU nodes
Added proper node selectors and tolerations
Enabled Prometheus ServiceMonitor integration

✅ Confirmed GPU metrics collection:

Verified DCGM Exporter deployment
Accessed the metrics endpoint
Validated GPU telemetry data

Next Steps

In the following sections, we will:

Configure Grafana dashboards for GPU monitoring
Learn how to monitor LLM inference workloads using these tools for both vLLM and Ray
You can now proceed to the next module to learn about setting up custom dashboards for monitoring your GPU workloads.

