---
title: Observability
---

# Observability

Pixee Enterprise Server provides comprehensive observability tools to help you monitor and understand your deployment's behavior.

## Available Tools

### Metrics & Dashboards

Monitor real-time metrics and visualize system performance with custom dashboards. Track AI service performance, per-finding task metrics, and more.

[Learn more about Metrics →](metrics.md)

### Logs & Debugging

Access pod status, view application logs, and query centralized logs with VictoriaLogs. When local metrics is enabled, logs from all Pixee components are automatically collected and searchable via the VictoriaLogs VMUI.

[Learn more about Logs & Debugging →](logs.md)

### Traces

Inspect distributed traces from the analysis service with VictoriaTraces. When local metrics is enabled, trace telemetry is automatically collected, allowing you to search by service name, trace ID, or span attributes.

[Learn more about Traces →](traces.md)

## Getting Started

For most operational tasks, you'll need:

- `kubectl` access to your cluster
- SSH access to the cluster host (for Embedded Cluster deployments)
- Knowledge of your deployment namespace:
  - **Embedded Cluster**: `kotsadm`
  - **Helm Deployment**: `pixee-enterprise-server`
