# Grafana Dashboard Creation Guide

This guide explains how to create effective dashboards in Grafana, specifically tailored for your multi-cluster setup.

---

## 🏗️ 1. Create a New Dashboard

1.  Log in to Grafana: [https://grafana.monitoring.opsmx.org](https://grafana.monitoring.opsmx.org)
2.  In the left sidebar, click the **+ (Create)** icon and select **Dashboard**.
3.  Click **Add a new panel**.

---

## 📊 2. Creating a Visualization (Panel)

To create a panel that filters data by cluster:

1.  **Data Source**: Select **Prometheus**.
2.  **Query**: Enter a Prometheus query. 
    
    > [!IMPORTANT]  
    > **Always use double quotes** around label values (e.g., `"target-cluster-01"`). Prometheus will fail with a "parse error: unexpected identifier" if you omit them.

    *   *Correct*: `sum(up{cluster="target-cluster-01"})`
    *   *Incorrect*: `sum(up{cluster=target-cluster-01})`
3.  **Visualization**: Choose a graph type (Time series, Bar gauge, etc.) from the right sidebar.
4.  **Title**: Set a descriptive title in the "Panel options" section.
5.  **Apply**: Click **Apply** in the top right corner.

---

## 🔄 3. Using Variables (Highly Recommended)

Variables allow you to create a single dashboard that can switch between different clusters instantly.

### Step 1: Create the Cluster Variable
1.  In your dashboard, click the **Dashboard settings** (cog icon) in the top right.
2.  Go to **Variables** > **Add variable**.
3.  **Name**: `cluster`
4.  **Type**: `Query`
5.  **Data source**: `Prometheus`
6.  **Query**: `label_values(up, cluster)`
7.  **Multi-value**: Enable this if you want to see data from multiple clusters at once.
8.  Click **Apply**.

### Step 2: Use the Variable in Panels
Update your panel queries to use the variable instead of a hardcoded string:
*   **Dynamic Query**: `up{cluster="$cluster"}`

Now, a dropdown menu will appear at the top of your dashboard allowing you to pick `target-cluster-01`, `local`, etc.

---

## 📥 4. Importing Community Dashboards

Instead of building from scratch, you can import professional dashboards from Grafana Labs:

1.  Go to **Create > Import** (or Dashboard > New > Import).
2.  You have two options for JSON:
    *   **Upload JSON file**: Click the **Upload JSON file** button and select your `.json` file.
    *   **Paste JSON**: Paste the JSON code directly into the **Import via panel json** text area.
3.  Click **Load**.
4.  Select your **Prometheus** data source in the dropdown.
5.  Click **Import**.

### Dashboard IDs for Search:
If you don't have a JSON file, you can enter these IDs from [grafana.com/dashboards](https://grafana.com/dashboards):
*   **ID 1860**: Node Exporter Full (Host metrics)
*   **ID 15760**: Kubernetes / Compute Resources / Namespace (Pods)

---

---

## 🚀 6. Useful Multi-Cluster Queries

Copy and paste these directly into your panels. Remember to replace `target-cluster-01` with your actual cluster name variable `$cluster` later if you set up variables.

### Cluster Health (Is it up?)
```promql
up{cluster="target-cluster-01"}
```

### Total CPU Usage (All Pods)
```promql
sum(rate(container_cpu_usage_seconds_total{cluster="target-cluster-01", container!=""}[5m]))
```

### Memory Usage Percentage (Per Node)
```promql
100 * (1 - (node_memory_MemAvailable_bytes{cluster="target-cluster-01"} / node_memory_MemTotal_bytes{cluster="target-cluster-01"}))
```

### Count of Running Pods
```promql
sum(kube_pod_status_phase{cluster="target-cluster-01", phase="Running"})
```

### Networking: Bytes Received (Per Node)
```promql
sum by (instance) (rate(node_network_receive_bytes_total{cluster="target-cluster-01", device!~"veth.*"}[5m]))
```

---

## 🏥 7. Pod Health & Troubleshooting Queries

If you suspect pods are not running correctly, use these queries to find the exact problematic pods.

### Find Pods that are NOT "Running"
This will list all pods in states like `Pending`, `Failed`, or `Succeeded` (completed).
```promql
kube_pod_status_phase{cluster="target-cluster-01", phase!="Running"} > 0
```

### Identify Crashing Pods (High Restarts)
This shows pods that have restarted in the last hour.
```promql
increase(kube_pod_container_status_restarts_total{cluster="target-cluster-01"}[1h]) > 0
```

### Find Pods in "ContainerCreating" or "ImagePullBackOff"
Check the `kube_pod_container_status_waiting_reason` metric:
```promql
kube_pod_container_status_waiting_reason{cluster="target-cluster-01"} > 0
```

### List Pods by Namespace Table
If you want to see a table of all pods and their current status:
- **Query**: `kube_pod_status_phase{cluster="target-cluster-01", phase="Running"} == 1`
- **Format**: Select **Table** visualization in Grafana.
- **Transform**: Go to the **Transform** tab and select **Labels to fields** to see the pod names and namespaces as columns.
