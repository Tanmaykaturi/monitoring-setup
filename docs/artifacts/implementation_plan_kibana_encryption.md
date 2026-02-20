# Kibana Alerting Encryption Key Setup Plan

This plan outlines the steps to configure the necessary encryption keys in Kibana to enable the Alerting and Actions features.

## Proposed Changes

### [Monitoring Cluster]

#### 1. Generate Encryption Keys
Generate three unique 32-character keys for:
- `xpack.encryptedSavedObjects.encryptionKey`
- `xpack.reporting.encryptionKey`
- `xpack.security.encryptionKey`

#### 2. Update Kibana Deployment
Add these keys as environment variables to the `kibana-kibana` deployment in the `logging` namespace using `kubectl set env`. This method ensures the keys are applied to the running pods immediately without needing to manually edit YAML files.

```bash
kubectl --kubeconfig central-efk.config set env deployment/kibana-kibana -n logging \
  XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=<KEY> \
  XPACK_REPORTING_ENCRYPTIONKEY=<KEY> \
  XPACK_SECURITY_ENCRYPTIONKEY=<KEY>
```

## Verification Plan

### Automated Tests
1. **Verify Environment Variables**:
   ```bash
   kubectl --kubeconfig central-efk.config get deployment kibana-kibana -n logging -o jsonpath='{.spec.template.spec.containers[0].env}'
   ```
2. **Check Kibana Pod Logs**:
   - Ensure the pod starts correctly without encryption key warnings.
   - `kubectl --kubeconfig central-efk.config logs -l app=kibana -n logging --tail=100`

### Manual Verification
1. Log in to Kibana.
2. Navigate to **Kibana > Observability > Alerts**.
3. Verify that the "Additional setup required" message is gone and the "Create rule" button is active.
