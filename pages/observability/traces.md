---
title: Traces
---

# Traces

Pixee Enterprise Server includes VictoriaTraces for distributed trace collection. When local metrics is enabled, trace telemetry from the analysis service is automatically collected and stored, allowing you to inspect individual request spans and trace IDs.

## Enabling Traces

Traces are automatically collected when local metrics is enabled. See [Enabling Local Metrics](metrics.md#enabling-local-metrics) for instructions.

## Accessing Traces VMUI

You can access the VictoriaTraces web interface to search and explore traces. You can either enable web access via ingress or use port forwarding.

### Option 1: Enable VMUI Web Interface (Ingress)

Enable the VMUI web interface to access the traces query interface directly through your browser.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To enable the VictoriaTraces web interface in Embedded Cluster deployments:

1. Navigate to the admin console
2. Select the `Config` tab
3. Go to the `Advanced Settings` section
4. Check the `Enable Traces VMUI web interface` option
5. Save and redeploy the application

Once enabled, access the traces interface at:

<CommandBlock>
https://<your-domain>/traces/select/vmui/
</CommandBlock>

<Warning title="Unauthenticated Access">
The VMUI web interface endpoints are not authenticated. Only enable this option if your deployment is within a trusted network or you have implemented external authentication.
</Warning>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To enable the VictoriaTraces web interface in Helm Deployment, add the following to your `values.yaml`:

<CommandBlock>
victoriatraces:
  server:
    ingress:
      enabled: true
      ingressClassName: "nginx" # Use your ingress class
      hosts:
        - name: "your-domain.com"
          path:
            - /traces
          port: http
</CommandBlock>

Then upgrade your deployment:

<CommandBlock>
helm upgrade pixee-enterprise-server ./charts/pixee-enterprise-server \
  -f values.yaml \
  -n pixee-enterprise-server
</CommandBlock>

Once enabled, access the traces interface at:

<CommandBlock>
https://your-domain.com/traces/select/vmui/
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

<CommandBlock>
ssh -L 10428:localhost:10428 pixee@<your-hostname>
</CommandBlock>

**Step 2: Set up port forwarding**

In the SSH session, run:

<CommandBlock>
sudo kubectl port-forward svc/pixee-enterprise-server-traces-server 10428:10428 -n kotsadm
</CommandBlock>

**Step 3: Access the traces interface**

Open your browser and navigate to:

<CommandBlock>
http://localhost:10428/traces/select/vmui/
</CommandBlock>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

**Step 1: Port forward to VictoriaTraces**

<CommandBlock>
kubectl port-forward svc/pixee-enterprise-server-traces-server 10428:10428 -n pixee-enterprise-server
</CommandBlock>

**Step 2: Access the traces interface**

Open your browser and navigate to:

<CommandBlock>
http://localhost:10428/traces/select/vmui/
</CommandBlock>

{{/if}}

## Querying Traces

VictoriaTraces provides a Jaeger-compatible query API for searching traces. In the VMUI, you can:

- **Search by service name** — Filter traces from specific services (e.g., `pixee-analysis-service`)
- **Search by trace ID** — Look up a specific trace using its trace ID
- **Filter by duration** — Find slow requests by setting minimum/maximum duration
- **Filter by tags** — Search for traces with specific span attributes

## VictoriaTraces Resources

- [VictoriaTraces Documentation](https://docs.victoriametrics.com/victoriatraces/)
- [VictoriaTraces Querying](https://docs.victoriametrics.com/victoriatraces/querying/) - Query API reference
