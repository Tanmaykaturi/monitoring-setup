# Debugging ELK 404 & Missing Data View

- [x] Investigate 404 error in Fluent-bit logs <!-- id: 54 -->
- [x] Verify Ingress resources on Monitoring Cluster <!-- id: 55 -->
- [x] Fix missing `elasticsearch-external` ingress <!-- id: 56 -->
- [x] Resolve "broken connection" (502) error in Fluent-bit <!-- id: 57 -->
    - [x] Identify cause: Ingress protocol mismatch (HTTP vs HTTPS) <!-- id: 60 -->
    - [x] Update Ingress protocol to HTTPS and disable SSL verification <!-- id: 61 -->
- [x] Perform Fluent-bit scratch installation on Target Cluster <!-- id: 62 -->
    - [x] Cleanup existing pods and DaemonSet <!-- id: 63 -->
    - [x] Retrieve latest `elastic` password from Central Cluster <!-- id: 64 -->
    - [x] Install Fluent-bit with verified `values.yaml` <!-- id: 65 -->
- [x] Resolve "400 Bad Request" (Unknown parameter _type) error <!-- id: 66 -->
    - [x] Identify cause: Deprecated `_type` parameter in ES 8+ <!-- id: 67 -->
    - [x] Update `values.yaml` with `Suppress_Type_Name On` <!-- id: 68 -->
- [x] Verify index creation in Kibana <!-- id: 58 -->
- [x] Create/update Data View <!-- id: 59 -->
