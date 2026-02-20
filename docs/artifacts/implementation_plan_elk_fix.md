# Fix 502 Bad Gateway for External Elasticsearch Access

The `elasticsearch-external` ingress on the Monitoring Cluster is currently sending HTTP (plaintext) traffic to the Elasticsearch service, which is configured to require HTTPS. This results in a `502 Bad Gateway` error and causing Fluent-bit on the target cluster to enter a `CrashLoopBackOff`.

## Proposed Changes

### [Monitoring Cluster]

#### [MODIFY] [ingress-elasticsearch-external.yaml](file:///home/admins/Downloads/elk-kibana/ingress-elasticsearch-external.yaml)
Update the Ingress annotations to use the HTTPS backend protocol and disable SSL verification for the backend (as it likely uses self-signed certificates).

```diff
   annotations:
     kubernetes.io/ingress.class: nginx
     nginx.ingress.kubernetes.io/proxy-body-size: "100m"
     nginx.ingress.kubernetes.io/proxy-buffer-size: "128k"
+    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
+    nginx.ingress.kubernetes.io/proxy-ssl-verify: "off"
```

## Verification Plan

### Automated Tests
1. **Apply the change**:
   ```bash
   kubectl --kubeconfig central-efk.config apply -f ingress-elasticsearch-external.yaml
   ```
2. **Check Fluent-bit logs on Target Cluster**:
   ```bash
   kubectl --kubeconfig /home/admins/Downloads/kubeconfig/ssd-use.config logs -l app.kubernetes.io/name=fluent-bit -n logging -f
   ```
   *Expected*: The `502` errors should disappear, and logs should show successful connection to the host on port 443.

### Manual Verification
1. Go to Kibana (**Stack Management > Data Views**) and check if the indices (e.g., `target-01-logs-*`) are now visible.
