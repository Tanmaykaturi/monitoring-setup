# Multi-Cluster Alerting Implementation Plan

This plan outlines the steps to configure centralized alerting for both Metrics (Prometheus) and Logs (ELK) in your multi-cluster environment.

## Proposed Changes

### [Monitoring Cluster]

#### [NEW] [alerting_guide.md](file:///home/admins/.gemini/antigravity/brain/405017a8-f49b-4ecf-8240-d8829344cbed/alerting_guide.md)
Create a comprehensive guide for setting up and managing alerts across multiple clusters.

#### [MODIFY] [alerts.yaml](file:///home/admins/Downloads/elk-kibana/alerts.yaml)
Update existing alert rules to include the `cluster` label and add new rules for monitoring the health of the remote write connection.

```diff
-      expr: up{job="node-exporter"} == 0
+      expr: up{job="node-exporter"} == 0
+      # The 'cluster' label will be automatically inherited from the metric
```

### [Kibana/ELK]
Documentation only (as Kibana alerts are typically created via the UI):
- Provide instructions for creating "Elasticsearch Query" alerts for log patterns (e.g., "Critical" logs from `target-01`).

## Verification Plan

### Automated Tests
1. **Apply Prometheus Rules**:
   ```bash
   kubectl --kubeconfig central-efk.config apply -f alerts.yaml
   ```
2. **Verify Rules in Prometheus UI**:
   - Access Prometheus UI -> Status -> Rules.
   - Confirm new rules are active and show the `cluster` label in the expression/results.

### Manual Verification
1. **Simulate a failure**: Manually stop a service on the target cluster and verify that an alert is triggered in the Prometheus "Alerts" tab with the correct `cluster="target-cluster-01"` label.
2. **Kibana Alert Test**: Create a test alert in Kibana (as per the guide) and use the "Run" feature to verify it detects matching log entries.
