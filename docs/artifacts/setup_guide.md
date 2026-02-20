# Infrastructure Setup Guide: Monitoring & Logging

This guide details the step-by-step process used to set up the Monitoring (Prometheus/Grafana) and Logging (ELK) stacks on the Kubernetes cluster.

---

## 🏗 Phase 1: Common Infrastructure

### 1. Namespace Creation
Separate namespaces were created to isolate the workloads.
```bash
kubectl create namespace monitoring
kubectl create namespace logging
```

### 2. Ingress Controller Setup
The Nginx Ingress Controller was installed to handle external traffic.
```bash
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

---

## 📊 Phase 2: Monitoring Stack (Prometheus & Grafana)

The `kube-prometheus-stack` was used for a complete monitoring solution.

### 1. Installation
```bash
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --version 81.6.7
```

### 2. Custom Alert Rules
The `alerts.yaml` file was applied to define cluster-specific alerts (NodeDown, HighCPUUsage, etc.).
```bash
kubectl apply -f alerts.yaml -n monitoring
```

---

## 🪵 Phase 3: ELK Stack (Logging)

The logging stack was deployed using a combination of Helm charts and custom manifests.

### 1. Elasticsearch Installation
Elasticsearch was installed with a soft anti-affinity policy to ensure it can run on the available nodes while maintaining 3 replicas.
```bash
helm upgrade --install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --version 8.5.1 \
  -f es-values.yaml
```

### 2. Fluent-bit Installation
Fluent-bit was deployed to collect and ship logs to Elasticsearch.
```bash
helm upgrade --install fluent-bit fluent/fluent-bit \
  --namespace logging \
  --version 0.55.0
```

### 3. Kibana Deployment
Kibana was deployed using custom manifests and secrets for secure connection to Elasticsearch.
```bash
# Apply secrets first
kubectl apply -f kibana-user-secret.yaml -n logging
kubectl apply -f kibana-token-secret.yaml -n logging

# Deploy Kibana
kubectl apply -f kibana-deploy.yaml -n logging
```

---

## 🌐 Phase 4: External Access (Ingress)

Final ingress rules were applied to expose the internal services via the `opsmx.org` domain.

```bash
kubectl apply -f ingress-prometheus.yaml -n monitoring
kubectl apply -f ingress-grafana.yaml -n monitoring
kubectl apply -f ingress-kibana.yaml -n logging
```

---

## ✅ Phase 5: Verification

After deployment, connectivity was verified using the following steps:
1. **Pod Status**: Ensuring all pods in `monitoring` and `logging` are `Running`.
2. **Service Discovery**: Verifying Grafana can query Prometheus via the internal service URL.
3. **Log Flow**: Confirming that application logs are visible in the Kibana Discovery dashboard.
4. **Endpoint Access**: Testing the external URLs in a browser.

---

> [!TIP]
> Always ensure the `central-efk.config` (kubeconfig) is active before running these commands to target the correct cluster.
