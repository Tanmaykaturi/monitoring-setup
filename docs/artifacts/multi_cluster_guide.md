# Detailed Multi-Cluster Setup Guide: Metrics & Logs

This guide provides the exact configuration steps to connect a **Target Cluster** to your central **Monitoring Cluster**.

---

## 📊 1. Metrics: Prometheus Remote Write

### Phase A: Configure Monitoring Cluster (The Receiver)

1. **Update Helm Values**: Create a file named `monitoring-receiver-values.yaml`:
   ```yaml
   prometheus:
     prometheusSpec:
       enableRemoteWriteReceiver: true
       # Optional: External URL if needed for certain proxies
       externalUrl: https://prometheus.monitoring.opsmx.org
   ```

2. **Upgrade Prometheus**:
   ```bash
   helm upgrade prometheus prometheus-community/kube-prometheus-stack \
     -n monitoring \
     -f monitoring-receiver-values.yaml
   ```

3. **Verify Endpoint**: Ensure the endpoint `/api/v1/write` returns a `405 Method Not Allowed` (which is correct for a GET request) when accessed externally:
   ```bash
   curl -i https://prometheus.monitoring.opsmx.org/api/v1/write
   ```

### Phase B: Configure Target Cluster (The Sender)

1. **Install Prometheus (Agent Mode)**: If the cluster doesn't have Prometheus, install it using the agent mode for efficiency.
   ```bash
   helm install prometheus-agent prometheus-community/prometheus \
     --set server.extraArgs.enable-feature=agent \
     --namespace monitoring --create-namespace
   ```

2. **Configure Remote Write**: Add this to a new file called `prometheus-agent-values.yaml` on the **Target Cluster**, then apply it.
   
   **The Configuration (`prometheus-agent-values.yaml`):**
   ```yaml
   extraConfigmapMounts:
     - name: remote-write-config
       mountPath: /etc/prometheus/remote_write
       configMap: prometheus-remote-write
       readOnly: true

   server:
     remoteWrite:
       - url: "https://prometheus.monitoring.opsmx.org/api/v1/write"
         write_relabel_configs:
           - target_label: "cluster"
             replacement: "target-cluster-01" # Unique ID for this specific cluster
   ```

   **The Command:**
   ```bash
   helm upgrade --install prometheus-agent prometheus-community/prometheus \
     --namespace monitoring \
     -f prometheus-agent-values.yaml
   ```

---

## 🪵 2. Logs: ELK External Shipping

### Phase A: Configure Monitoring Cluster (The Receiver)

### Phase A: Configure Monitoring Cluster (The Receiver)

1. **Expose Elasticsearch Service**: Apply this Ingress to allow external Fluent-bit instances to connect. 
   
   > [!NOTE]
   > We use `proxy-body-size` and `proxy-buffer-size` to handle large batches of logs from remote clusters without dropping them.

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: elasticsearch-external
     namespace: logging
     annotations:
       kubernetes.io/ingress.class: nginx
       nginx.ingress.kubernetes.io/proxy-body-size: "100m" # Support large log batches
       nginx.ingress.kubernetes.io/proxy-buffer-size: "128k"
       # Recommended: IP Whitelisting (See Security Section below)
   spec:
     rules:
     - host: elasticsearch.monitoring.opsmx.org
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: elasticsearch-master
               port:
                 number: 9200
   ```

2. **Retrieve Credentials**: You will need the `elastic` user password for the Target Cluster to authenticate.
   ```bash
   kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath="{.data.password}" | base64 --decode
   ```

### Phase B: Configure Target Cluster (The Sender)

#### 1. Prepare Environment
Add the official Fluent Helm repository and create the namespace:
```bash
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
kubectl create namespace logging
```

#### 2. Create Configuration (`fluent-bit-values.yaml`)
Create this file to define both the **Cluster Labeling** and the **Remote Output**.

```yaml
config:
  service: |
    [SERVICE]
        Flush         1
        Log_Level     info
        Daemon        off
        Parsers_File  parsers.conf

  # STEP 1: Identification - Label every log with the cluster name
  filters: |
    [FILTER]
        Name          modify
        Match         *
        Add           cluster target-cluster-01

  # STEP 2: Shipping - Send to Central Elasticsearch
  outputs: |
    [OUTPUT]
        Name            es
        Match           *
        Host            elasticsearch.monitoring.opsmx.org
        Port            443
        TLS             On
        TLS.Verify      Off
        HTTP_User       elastic
        HTTP_Passwd     <PASSWORD_FROM_RECEIVER_PHASE_A>
        Logstash_Format On
        Logstash_Prefix target-01-logs
        Buffer_Size     5M
```

#### 3. Install from Scratch
Run the installation command using the values file:
```bash
helm install fluent-bit fluent/fluent-bit \
  --namespace logging \
  -f fluent-bit-values.yaml
```

---

## 🛡️ 3. Security Best Practices

> [!WARNING]
> Exposing Prometheus and Elasticsearch to the internet is high-risk. Follow these hardening steps:

1. **IP Whitelisting**: Restrict Ingress access to the NAT IP of the Target Cluster.
   ```yaml
   annotations:
     nginx.ingress.kubernetes.io/whitelist-source-range: "target_cluster_ip/32"
   ```
2. **Mutual TLS (mTLS)**: If possible, use client certificates for authentication between clusters.
3. **Basic Auth**: At a minimum, use Nginx Basic Auth on the Ingress if the application layer doesn't have sufficient protection.

---

## ✅ 4. Verification & Troubleshooting

Follow these steps to confirm your setup is working correctly.

### 📊 Verification: Metrics (Prometheus)

1.  **Check Monitoring Cluster (Grafana UI):**
    *   Go to **Explore** in your Grafana dashboard.
    *   Select the **Prometheus** datasource.
    *   Run this query: `up{cluster="target-cluster-01"}`
    *   **Success**: You should see entries for the nodes/pods of your Target Cluster.

2.  **Check Target Cluster (Logs):**
    *   If metrics are missing, check the logs of the Prometheus server on the **Target Cluster**:
    *   `kubectl logs -l app=prometheus,component=server -n monitoring`
    *   Look for errors containing "Remote write" or HTTP status codes like 4xx/5xx.

### 🪵 Verification: Logs (ELK)

1.  **Check Monitoring Cluster (Kibana UI):**
    *   Go to **Management > Stack Management > Data Views (or Index Patterns)**.
    *   Click **Create data view**.
    *   Check if `logs-target-01-*` (or the prefix you chose) appears in the "Source" list.
    *   **Success**: If the index appears, data is flowing.

2.  **Check Target Cluster (Fluent-bit Logs):**
    *   `kubectl logs -l app.kubernetes.io/name=fluent-bit -n logging`
    *   Look for messages like `[target_01] connection established` or errors like `[error] [output:es:es.1] error query resource`.

---

## 🛠️ Common Troubleshooting

| Issue | Potential Cause | Fix |
| :--- | :--- | :--- |
| **403 Forbidden** | Ingress Whitelist | Ensure Target Cluster's NAT IP is in the whitelist annotation. |
| **504 Timeout** | Firewall/Network | Ensure Port 443 is open from Target Cluster to Monitoring Cluster IP. |
| **Certificate Error** | TLS verification | Use `TLS.Verify Off` temporarily to test, or add the Monitoring Cluster's CA to the Target Cluster. |
| **No Metrics** | Receiver Disabled | Ensure `enableRemoteWriteReceiver: true` is set in Monitoring Cluster values. |
