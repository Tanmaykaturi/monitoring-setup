# Multi-Cluster Monitoring & Logging (SSD Project)

This repository contains the full configuration and documentation for the centralized monitoring (Prometheus/Grafana) and logging (ELK) stack, designed for a multi-cluster Kubernetes environment.

## 📖 In-Depth Documentation
The primary source of truth for this project is the **[Master Documentation](master_documentation.md)**, which covers:
- Complete end-to-end setup (Receivers & Senders)
- Security hardening (Kibana encryption keys)
- Dashboards and Alerting configurations
- Troubleshooting common errors (502, 401, 400)

## 📂 Repository Structure
- `/`: Main configuration manifests (Ingress, Deployments, Values).
- `master_documentation.md`: The combined final technical guide.
- `docs/artifacts/`: All individual research, implementation plans, and sub-guides generated during development.

## ⚙️ Core Manifests
- `ingress-kibana.yaml`: External access for Kibana.
- `ingress-elasticsearch-external.yaml`: Secure log intake from remote clusters.
- `alerts.yaml`: Cluster-aware Prometheus alerting rules.
- `fluent-bit-values-target.yaml`: Corrected and verified Fluent-bit configuration.

---

> [!CAUTION]
> **Security Warning**: This repository currently contains `central-efk.config`. Ensure this file is handled securely or moved to a secret management system before pushing to a public repository.
