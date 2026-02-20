# Project Infrastructure Documentation: Monitoring & Logging

This document provides a comprehensive overview of the Monitoring and Logging infrastructure implemented on the Kubernetes cluster.

---

## 📊 Monitoring Infrastructure (Prometheus & Grafana)

The monitoring stack is deployed in the `monitoring` namespace and provides real-time metrics, alerting, and visualization for the cluster components.

### 1. Components
- **Prometheus**: Collected via the `kube-prometheus-stack`. It scrapes metrics from nodes, pods, and the Kubernetes API.
- **Grafana**: Provides visual dashboards. It is pre-configured with a default datasource for Prometheus.
- **Alertmanager**: Handles alerting based on rules defined in Prometheus.
- **Node Exporter**: Deployed as a DaemonSet to collect hardware and OS metrics.

### 2. Configuration & Connectivity
- **Prometheus URL**: `http://prometheus-kube-prometheus-prometheus.monitoring:9090`
- **Grafana URL**: [https://grafana.monitoring.opsmx.org](https://grafana.monitoring.opsmx.org)
- **Automatic Connection**: The Grafana instance is connected to Prometheus via a ConfigMap-defined datasource (`prometheus-kube-prometheus-grafana-datasource`).

### 3. Alerting Rules
Custom alert rules are defined in `alerts.yaml`, including:
- **NodeDown**: Critical alert if a node-exporter job is down.
- **HighCPUUsage**: Warning alert if CPU usage exceeds 80% for 5 minutes.
- **KubeAPIDown**: Critical alert if the Kubernetes API server is unreachable.

---

## 🪵 Logging Infrastructure (ELK Stack)

The logging stack (Elasticsearch, Logstash/Fluent-bit, Kibana) is deployed in the `logging` namespace to centralize logs from all applications and system components.

### 1. Components
- **Elasticsearch**: The distributed search and analytics engine. Deployed with 3 replicas for high availability (configured in `es-values.yaml`).
- **Kibana**: The visualization layer for Elasticsearch logs.
- **Fluent-bit**: Acting as the log collector/shipper to send logs from pods to Elasticsearch.

### 2. Configuration & Access
- **Kibana URL**: [https://kibana.monitoring.opsmx.org](https://kibana.monitoring.opsmx.org)
- **Authentication**:
  - Kibana connects to Elasticsearch using the `kibana_system` user.
  - Credentials are managed via Kubernetes Secrets (`elasticsearch-master-credentials` and `kibana-elasticsearch-credentials`).
- **Storage & Replicas**: Elasticsearch is configured with `antiAffinity: "soft"` to allow scheduling multiple replicas on the same node if necessary for development/testing, while maintaining 3-node cluster health at the software level.

### 3. Ingress Management
Traffic to both Kibana and Grafana is managed by Nginx Ingress Controllers, sharing the same external IP (`146.20.63.99`) but using host-based routing.

---

## 💾 SSD Configuration Details

I have verified the SSD-related configurations across both your specific clusters.

### 1. Monitoring Cluster (`central-efk.config`)
- **Storage Classes**: 
  - `ssdv2`: Standard SSD storage.
  - `ssdv2-performance`: High-performance SSD storage (Default).
- **Monitoring**:
  - `PrometheusRule/ssd-alerts`: Specifically monitors cluster health with SSD-related identifiers.

### 2. Second Cluster (`ssd-use.config`)
- **Storage Classes**:
  - `ssd`: Standard SSD storage (Default).
  - `ssd-large`: Larger capacity SSD storage.
- **Namespaces**: `ssd-tf`, `ssd-uat-jan`.
- **Metadata**: Several resources carry the `ssd.opsmx.com` label, specifically for internal token management.

---

## 🛠 Management & Troubleshooting

### Kubeconfig
The cluster is managed using the `central-efk.config` file.
```bash
export KUBECONFIG=central-efk.config
```

### Useful Commands
- **Check Status**: `kubectl get pods -n monitoring` / `kubectl get pods -n logging`
- **View Logs**: `kubectl logs <pod-name> -n <namespace>`
- **Check Ingress**: `kubectl get ingress -A`

---

> [!NOTE]
> All certificates and sensitive tokens are managed via Kubernetes Secrets in their respective namespaces. Ensure backups of these secrets are maintained according to security policies.
