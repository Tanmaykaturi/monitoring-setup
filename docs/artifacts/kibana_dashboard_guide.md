# Kibana Dashboard Guide: SSD Log Monitoring

This guide provides step-by-step instructions for creating a Kibana dashboard to track log volume, errors, and anomalies across your SSD environments and clusters.

---

## 🏗️ 1. Create a Data View (Index Pattern)
Before creating visualizations, Kibana needs to know which indices to look at.

1.  Log in to Kibana: [https://kibana.monitoring.opsmx.org](https://kibana.monitoring.opsmx.org)
2.  Go to **Stack Management > Data Views**.
3.  Click **Create data view**.
4.  **Name**: `SSD Logs`
5.  **Index pattern**: `logs-target-*` (or the specific index name used in your setup).
6.  **Timestamp field**: `@timestamp`.
7.  Click **Save data view to Kibana**.

---

## 🔍 2. Identify SSD Log Patterns & Error Types
Use the **Discover** tab to find these critical SSD-specific issues:

1.  **Provisioning Failures**: Search for `message : "VolumeBinding"` or `message : "Insufficient memory"`.
2.  **I/O Errors**: Search for `message : "Input/output error"` or `message : "Slow disk"`.
3.  **Storage Class Issues**: Search for `message : "storageclass.storage.k8s.io not found"`.
4.  **Application Criticals**: Search for `level : "error"` or `level : "critical"`.

---

## 📊 3. Create Visualizations (Lens)
Go to **Analytics > Dashboard** and click **Create new dashboard**, then **Create visualization**.

### A. Log Volume Trend (Line Chart)
*   **Action**: Drag the `@timestamp` field to the middle.
*   **Result**: Displays the number of logs over time.
*   **Filter**: Add a filter for `cluster : "target-cluster-01"` to see it per cluster.

### B. Error Count Breakdown (Bar Chart)
*   **Action**: Drag `level` to the workspace.
*   **Change Series**: Set the breakdown to the `kubernetes.container_name` field.
*   **Filter**: Add a filter where `level` is `error`.
*   **Result**: Shows which service is generating the most errors.

### C. Top Error Messages (Table)
*   **Action**: Select **Table** as the chart type.
*   **Rows**: Add the `message` field (you may need to use `message.keyword`).
*   **Filter**: `level : "error"`.
*   **Result**: A list of the most frequent error messages for L1/L2 teams to review.

---

## 🧩 4. Assemble and Configure Dashboard

1.  **Arrange**: Drag and resize the panels to create a clear layout.
2.  **Add Controls**: Click **Controls** at the top to add a "Dropdown" control.
    *   **Field**: `cluster` (This allows switching between SSD environments instantly).
3.  **Save**: Click **Save** in the top right. Name it `SSD Infrastructure Overview`.

---

## 🔔 6. Integrate Dashboard Alerts
Alerting should be integrated directly into your monitoring workflow.

1.  **Add Alert Panel**: Add a **Markdown** panel to your dashboard with links to the **Rules and Connectors** page.
2.  **Create Rule from Lens**: Inside a visualization, you can often click the "..." menu and select **Create alert rule** to quickly set up a threshold for that metric.
3.  **Kibana Connectors**: Ensure you have a **Connector** (Email/Slack) set up under **Stack Management > Connectors** so alerts have a destination.

---

> [!TIP]
> **Dashboard Definition Documentation**: 
> - **Primary Index**: `logs-target-*`
> - **Critical Fields**: `level`, `message`, `kubernetes.pod_name`, `cluster`.
> - **Refresh Rate**: Set to "Auto-refresh: 1 minute" for real-time monitoring.
