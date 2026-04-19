---
title: Metrics & Dashboards
---

# Metrics & Dashboards

Pixee Enterprise Server includes Victoria Metrics for local metrics collection and visualization. Custom dashboards are automatically deployed to help monitor AI service performance and per-finding task metrics.

<h2 id="enabling-local-metrics">Enabling Local Metrics</h2>

Before accessing Metrics dashboards, you must enable local metrics collection in your deployment.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To enable local metrics collection in Embedded Cluster deployments:

1. Navigate to the admin console
2. Select the `Config` tab
3. Go to the `Advanced Settings` section
4. Check the `Enable Local Metrics` option
5. Save and redeploy the application

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To enable local metrics collection in Helm Deployment, add the following to your `values.yaml`:

<CodeBlock language="yaml">
global:
  pixee:
    localMetrics:
      enabled: true
</CodeBlock>

Then upgrade your deployment:

<CommandBlock language="bash">
helm upgrade pixee-enterprise-server ./charts/pixee-enterprise-server \
  -f values.yaml \
  -n pixee-enterprise-server
</CommandBlock>

{{/if}}

<h2 id="accessing-metrics-dashboards">Accessing Metrics Dashboards</h2>

After enabling local metrics, you can access the Metrics dashboards to view real-time metrics and custom dashboards. You can either enable web access via ingress or use port forwarding.

### Option 1: Enable VMUI Web Interface (Ingress)

Enable the VMUI web interface to access metrics dashboards directly through your browser without port forwarding.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To enable the VMUI web interface in Embedded Cluster deployments:

1. Navigate to the admin console
2. Select the `Config` tab
3. Go to the `Advanced Settings` section
4. Check the `Enable VMUI web interface` option
5. Save and redeploy the application

Once enabled, access the dashboards at:

<CommandBlock>
https://<your-domain>/metrics/vmui/#/dashboards
</CommandBlock>

<Warning title="Unauthenticated Access">
The VMUI web interface endpoints are not authenticated. Only enable this option if your deployment is within a trusted network or you have implemented external authentication.
</Warning>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To enable the VMUI web interface in Helm Deployment, add the following to your `values.yaml`:

<CommandBlock language="yaml">
victoria-metrics-single:
  server:
    ingress:
      enabled: true
      ingressClassName: "nginx" # Use your ingress class
      hosts:
        - name: "your-domain.com"
          path:
            - /metrics
          port: http
</CommandBlock>

Then upgrade your deployment:

<CommandBlock language="bash">
helm upgrade pixee-enterprise-server ./charts/pixee-enterprise-server \
  -f values.yaml \
  -n pixee-enterprise-server
</CommandBlock>

Once enabled, access the dashboards at:

<CommandBlock>
https://your-domain.com/metrics/vmui/#/dashboards
</CommandBlock>

<Warning title="Unauthenticated Access">
The VMUI web interface endpoints are not authenticated. Consider implementing external authentication or only enable this in trusted network environments.
</Warning>

{{/if}}

### Option 2: Port Forwarding

If you prefer not to expose the VMUI via ingress, you can use port forwarding for temporary access.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

**Step 1: Create SSH tunnel from your local machine**

<CommandBlock language="bash">
ssh -L 8428:localhost:8428 pixee@<your-hostname>
</CommandBlock>

**Step 2: Set up port forwarding**

In the SSH session, run:

<CommandBlock language="bash">
sudo kubectl port-forward pixee-enterprise-server-metrics-server-0 8428:8428 -n kotsadm
</CommandBlock>

**Step 3: Access the dashboards**

Open your browser and navigate to:

<CommandBlock>
http://localhost:8428/vmui/#/dashboards
</CommandBlock>

<Tip>
Keep the SSH tunnel and port-forward running while viewing the dashboards.
</Tip>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

**Step 1: Port forward to Metrics**

<CommandBlock language="bash">
kubectl port-forward pixee-enterprise-server-metrics-server-0 8428:8428 -n pixee-enterprise-server
</CommandBlock>

**Step 2: Access the dashboards**

Open your browser and navigate to:

<CommandBlock>
http://localhost:8428/vmui/#/dashboards
</CommandBlock>

{{/if}}

## Available Dashboards

Pixee Enterprise Server includes the following pre-configured dashboards:

### AI Service Metrics

Monitor AI service performance with the following metrics:

- **AI Service Requests Rate**: Track request volume by model, status, and status code
- **AI Service Latency Percentiles**: Monitor p50, p95, and p99 latency by model and status
- **AI Service Retry Rate**: Track retry frequency across services
- **AI Service Rate Limit Events**: Monitor rate limiting occurrences
- **AI Health Check Latency Percentiles**: Track health check performance (p50, p95, p99)

### Per-Finding Task Metrics

Track per-finding task execution with:

- **Per-Finding Task Counts by Type and Status**: Monitor task distribution and completion rates

### Example: Troubleshooting Analysis Latency

One of the most common use cases for the metrics dashboards is troubleshooting slow analysis performance. The **AI Service Latency Percentiles** dashboard is particularly useful for identifying which AI models are contributing to latency issues.

[![ai-latency-graph.png]({{asset "assets/metrics/ai-latency-graph.png"}})]({{asset "assets/metrics/ai-latency-graph.png"}})

**Understanding the Latency Percentiles Graph**

The AI Service Latency Percentiles dashboard displays three key metrics for each AI model over time:

- **p50 (Median)**: The median latency - half of all requests complete faster than this value, half complete slower. This represents typical performance.
- **p95**: 95% of requests complete faster than this value. This helps identify performance outliers while filtering out the worst 5%.
- **p99 (Maximum)**: 99% of requests complete faster than this value. This captures near-worst-case performance and helps identify extreme latency spikes.

Each AI model (e.g., o3-mini) has its own set of percentile lines on the graph, allowing you to compare performance across models and identify which models are experiencing latency issues.

**Troubleshooting Scenario**

If users report that analysis is taking longer than expected:

1. **Access the AI Service Latency Percentiles dashboard** following the steps in [Accessing Metrics Dashboards](#accessing-metrics-dashboards)

2. **Identify the time period** when the slowdown occurred using the time range selector in VMUI

3. **Compare latency across models**:
   - Look for models with elevated p50 values - this indicates consistently slow performance
   - Check for spikes in p95 or p99 values - this indicates intermittent latency issues
   - Compare current latency values to historical baselines to confirm degradation

4. **Correlate with other metrics**:
   - Check the **AI Service Requests Rate** dashboard to see if increased request volume is causing the latency
   - Review the **AI Service Rate Limit Events** dashboard to see if rate limiting is delaying requests
   - Examine the **AI Service Retry Rate** to identify if failed requests are causing delays

5. **Take action** based on findings:
   - If a specific model shows consistently high latency, consider switching to an alternative model or contacting the AI service provider
   - If rate limiting is occurring, adjust request rates or increase service quotas
   - If all models show elevated latency during specific time periods, investigate external factors (network issues, AI service outages, etc.)

<h2 id="dashboard-updates">Dashboard Updates</h2>

<Note title="Restarting Metrics After Updates">
When new dashboards are added or existing dashboards are updated during an upgrade, the Metrics pod must be restarted to load the changes.

<CommandBlock language="bash">
# For Embedded Cluster
kubectl rollout restart statefulset pixee-enterprise-server-victoria-metrics-single-server -n kotsadm

# For Helm Deployment
kubectl rollout restart statefulset pixee-enterprise-server-victoria-metrics-single-server -n pixee-enterprise-server
</CommandBlock>

The pod will perform a rolling restart, which may take a few minutes. You can monitor the restart progress:

<CommandBlock language="bash">
# For Embedded Cluster
kubectl rollout status statefulset pixee-enterprise-server-victoria-metrics-single-server -n kotsadm

# For Helm Deployment
kubectl rollout status statefulset pixee-enterprise-server-victoria-metrics-single-server -n pixee-enterprise-server
</CommandBlock>

</Note>

<Warning title="User-Created Dashboards Are Not Persisted">
Custom dashboards created through the Metrics UI are stored in browser localStorage and are not persisted to the cluster. They will be lost when:

- The pod restarts
- You clear your browser cache
- You access Metrics from a different browser or device

**To preserve custom dashboards:**

1. Export them as JSON files before pod restarts
2. Save the JSON files to your local filesystem
3. Re-import them after the pod restart
   </Warning>

## Victoria Metrics VMUI

Victoria Metrics provides a powerful UI (VMUI) for querying and visualizing metrics beyond the pre-configured dashboards.

### Accessing VMUI

Access the full VMUI interface at:

<CommandBlock>
http://localhost:8428/vmui/
</CommandBlock>

### VMUI Features

From the VMUI, you can:

- **Execute PromQL queries**: Write custom queries to explore your metrics
- **Create custom visualizations**: Build charts and graphs for any metric
- **Explore available metrics**: Browse all collected metrics and their labels
- **View and create dashboards**: Access pre-configured dashboards or create your own
- **Export data**: Download metrics data for offline analysis

### Useful PromQL Queries

Here are some example queries you can run in VMUI:

<CommandBlock>
# View all metric names
{__name__!=""}

# AI service request rate (last 5 minutes)
rate(ai_service_requests[5m])

# AI service latency by model
histogram_quantile(0.95, sum(rate(ai_service_latency_ms_bucket[5m])) by (le, model))

# Per-finding task counts by status
sum by(status) (per_finding_tasks)
</CommandBlock>

## Metrics Retention

Metrics are retained for **3 days** by default. This retention period balances observability needs with storage requirements.

### Adjusting Retention Period

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To adjust the retention period in Embedded Cluster deployments:

1. Navigate to the admin console
2. Select the `Config` tab
3. Go to the `Advanced Settings` section
4. Update the `Metrics retention period` field (e.g., 3d, 7d, 30d, 1y)
5. Save and redeploy the application

The new retention period will be applied automatically during the deployment.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To adjust the retention period in Helm Deployment, add the following to your `values.yaml`:

<CommandBlock language="yaml">
victoria-metrics-single:
  server:
    retentionPeriod: "7d" # Options: 3d, 7d, 30d, 1y, etc.
</CommandBlock>

Then upgrade your deployment:

<CommandBlock language="bash">
helm upgrade pixee-enterprise-server ./charts/pixee-enterprise-server \
  -f values.yaml \
  -n pixee-enterprise-server
</CommandBlock>

The new retention period will be applied automatically during the deployment.

{{/if}}

## Troubleshooting

### Port Forward Fails

If port forwarding fails, verify the pod is running:

<CommandBlock language="bash">
# For Embedded Cluster
kubectl get pod pixee-enterprise-server-metrics-server-0 -n kotsadm

# For Helm Deployment
kubectl get pod pixee-enterprise-server-metrics-server-0 -n pixee-enterprise-server
</CommandBlock>

If the pod is not running, check the pod logs:

<CommandBlock language="bash">
# For Embedded Cluster
kubectl logs pixee-enterprise-server-metrics-server-0 -n kotsadm

# For Helm Deployment
kubectl logs pixee-enterprise-server-metrics-server-0 -n pixee-enterprise-server
</CommandBlock>

### Dashboards Not Appearing

If dashboards don't appear after an upgrade:

1. Verify local metrics is enabled (see [Enabling Local Metrics](#enabling-local-metrics))
2. Restart the Metrics pod (see [Dashboard Updates](#dashboard-updates))
3. Clear your browser cache and refresh the page
4. Check that you're accessing the correct URL: `http://localhost:8428/vmui/#/dashboards`

### No Metrics Data

If dashboards show no data:

1. Verify local metrics has been enabled for at least a few minutes (metrics need time to accumulate)
2. Ensure you have run at least one Pixee analysis to generate metrics data (trigger a repository scan, PR analysis, or other Pixee operation)
3. Check that the analysis and platform services are running and processing work
4. Verify the time range selector in VMUI is set appropriately (default is "Last 30 minutes")

## Additional Resources

- <a href="https://docs.victoriametrics.com/" target="_blank" rel="noopener noreferrer">VictoriaMetrics Documentation</a>
- <a href="https://docs.victoriametrics.com/metricsql/" target="_blank" rel="noopener noreferrer">MetricsQL Query Language</a> - VictoriaMetrics query language (PromQL-compatible with extensions)
- <a href="https://docs.victoriametrics.com/victoriametrics/vmui/" target="_blank" rel="noopener noreferrer">VMUI Documentation</a> - VictoriaMetrics web UI guide
- <a href="https://prometheus.io/docs/prometheus/latest/querying/examples/" target="_blank" rel="noopener noreferrer">PromQL Query Examples</a>
