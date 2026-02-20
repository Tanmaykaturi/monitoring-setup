# Multi-Cluster Alerting Guide: Metrics & Logs

This guide shows how to set up centralized alerting for your multi-cluster environment. Alerts are divided into **Metrics-based (Prometheus)** and **Log-based (ELK)**.

---

## 📊 1. Metrics Alerts (Prometheus)

Prometheus alerts are defined as `PrometheusRule` resources. Because we use Remote Write, your centralized Prometheus can see the `cluster` label for every alert.

### Step A: Define Alert Rules
Update your `alerts.yaml` to include cluster-aware alerts.

**Example: High Memory Usage (Multi-Cluster)**
```yaml
- alert: HighMemoryUsage
  expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High memory usage on {{ $labels.instance }} (Cluster: {{ $labels.cluster }})"
    description: "Memory usage is above 90% for more than 5 minutes."
```

### Step B: Monitor the Monitoring Connection
It is critical to know if a remote cluster stops sending metrics.

**Alert: Remote Write Failure**
```yaml
- alert: ClusterMetricsMissing
  expr: absent(up{cluster="target-cluster-01"})
  for: 10m
  labels:
    severity: critical
  annotations:
    summary: "Metrics from target-cluster-01 are missing!"
    description: "The central monitoring cluster has not received metrics from target-cluster-01 for 10 minutes."
```

---

## 📧 2. Notification Routing (Alertmanager)

Alertmanager handles **where** the alerts go (Email, Slack, etc.).

### Step A: Configure Receivers
Update `alertmanager-config.yaml` to define your L1 and L2 teams.

### Step B: Routing by Cluster
You can route alerts to different teams based on which cluster is failing.
```yaml
route:
  routes:
    - match:
        cluster: target-cluster-01
      receiver: target-01-admin-team
```

---

## 🪵 3. Log Alerts (Kibana)

Kibana alerts (Rules) allow you to trigger notifications based on log patterns.

### Step A: Configure a Connector
Before creating a rule, you need a destination:
1. Go to **Kibana > Stack Management > Connectors**.
2. Click **Create connector**.
3. Select **Email** or **Webhook/Slack**.
4. **Configuration**: Use the SMTP/Webhook details provided by your infrastructure team (e.g., matching the Alertmanager settings).

### Step B: Create an SSD SSD Log Rule
1. Go to **Kibana > Analytics > Discover** (with your `SSD Logs` data view).
2. Click **Alerts > Create search threshold rule**.
3. **Condition**:
   - `When count is > 50`
   - `For the last 5 minutes`
   - **Filter**: `level : "error" OR message : "SSD_PROVISION_FAILURE"`
4. **Action**: Select the **Connector** you created in Step A.
5. **Message**: Customize the message to include the `{{context.title}}` and a link back to the Dashboard.

### Step C: Anomaly Detection (Advanced)
If you have the Elastic ML license:
- Create a **Machine Learning Job** to detect "Log Rate Anomalies".
- Create an **Alert Rule** for when the job detects a "Critical" anomaly score.

---

## ✅ Best Practices
1.  **Avoid Alert Fatigue**: Use the `for` duration (e.g., `for: 5m`) so that brief spikes don't trigger unnecessary emails.
2.  **Use Labels**: Always include the `{{ $labels.cluster }}` and `{{ $labels.instance }}` in your annotations so the L1 team knows exactly which environment is broken.
3.  **Test Annually**: Periodically simulate a failure to ensure the notification path still works.
