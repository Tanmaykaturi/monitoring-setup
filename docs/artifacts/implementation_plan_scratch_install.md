# Fluent-bit Scratch Installation Plan (Target Cluster)

This plan outlines the steps to perform a clean, "from scratch" installation of Fluent-bit on the `ssd-use.config` cluster. This will resolve the persistent `CrashLoopBackOff`, protocol mismatch (fixed on ingress), and authentication errors.

## Proposed Changes

### [Monitoring Cluster]

#### 1. Retrieve Latest Credentials
Execute command to get the verified `elastic` user password.

### [Target Cluster]

#### 1. Cleanup existing resources
Remove the existing Fluent-bit installation to ensure a clean state.
```bash
helm uninstall fluent-bit -n logging
kubectl delete pods -n logging --field-selector status.phase=Failed
```

#### 2. Verify Ingress Availability
Perform a `curl` to the external Elasticsearch host from a temporary pod to confirm the network path is open.

#### 3. Install Fluent-bit
Perform a fresh installation using the verified `fluent-bit-values.yaml`.

## Verification Plan

### Automated Tests
1. **Retrieve Password**:
   ```bash
   kubectl --kubeconfig central-efk.config get secret elasticsearch-master-credentials -n logging -o jsonpath="{.data.password}" | base64 --decode
   ```
2. **Uninstall & Cleanup**:
   ```bash
   helm --kubeconfig /home/admins/Downloads/kubeconfig/ssd-use.config uninstall fluent-bit -n logging
   kubectl --kubeconfig /home/admins/Downloads/kubeconfig/ssd-use.config delete pods -n logging --field-selector status.phase=Failed
   ```
3. **Reinstall**:
   ```bash
   helm --kubeconfig /home/admins/Downloads/kubeconfig/ssd-use.config install fluent-bit fluent/fluent-bit -n logging -f fluent-bit-values.yaml
   ```
4. **Check Logs**:
   ```bash
   kubectl --kubeconfig /home/admins/Downloads/kubeconfig/ssd-use.config logs -l app.kubernetes.io/name=fluent-bit -n logging -f
   ```
   *Expected*: Logs should show `connection established` and successful data flushes.

### Manual Verification
1. Verify index creation in Kibana (**Stack Management > Data Views**).
2. Create Data View for `target-01-logs-*`.
