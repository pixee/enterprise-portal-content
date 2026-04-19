---
title: Logs & Debugging
---

# Logs & Debugging

Access pod status, view application logs, and query centralized logs for your Pixee Enterprise Server deployment.

## VictoriaLogs (Centralized Logging)

When local metrics is enabled, Pixee Enterprise Server also deploys VictoriaLogs for centralized log collection. VictoriaLogs automatically collects and indexes logs from all Pixee components, making it easy to search and analyze logs across your entire deployment.

### Enabling VictoriaLogs

VictoriaLogs is automatically enabled when you enable local metrics. See [Enabling Local Metrics](metrics.md#enabling-local-metrics) for instructions.

### Accessing VictoriaLogs VMUI

You can access the VictoriaLogs web interface to query and explore logs. You can either enable web access via ingress or use port forwarding.

#### Option 1: Enable VMUI Web Interface (Ingress)

Enable the VMUI web interface to access the logs query interface directly through your browser.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To enable the VictoriaLogs web interface in Embedded Cluster deployments:

1. Navigate to the admin console
2. Select the `Config` tab
3. Go to the `Advanced Settings` section
4. Check the `Enable VMUI web interface` option (this enables both metrics and logs interfaces)
5. Save and redeploy the application

Once enabled, access the logs interface at:

<CommandBlock>
https://<your-domain>/logs/vmui/
</CommandBlock>

<Warning title="Unauthenticated Access">
The VMUI web interface endpoints are not authenticated. Only enable this option if your deployment is within a trusted network or you have implemented external authentication.
</Warning>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To enable the VictoriaLogs web interface in Helm Deployment, add the following to your `values.yaml`:

<CommandBlock>
victoria-logs-single:
  server:
    ingress:
      enabled: true
      ingressClassName: "nginx" # Use your ingress class
      hosts:
        - name: "your-domain.com"
          path:
            - /logs
          port: http
</CommandBlock>

Then upgrade your deployment:

<CommandBlock>
helm upgrade pixee-enterprise-server ./charts/pixee-enterprise-server \
  -f values.yaml \
  -n pixee-enterprise-server
</CommandBlock>

Once enabled, access the logs interface at:

<CommandBlock>
https://your-domain.com/logs/vmui/
</CommandBlock>

<Warning title="Unauthenticated Access">
The VMUI web interface endpoints are not authenticated. Consider implementing external authentication or only enable this in trusted network environments.
</Warning>

{{/if}}

#### Option 2: Port Forwarding

If you prefer not to expose the VMUI via ingress, you can use port forwarding for temporary access.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

**Step 1: Create SSH tunnel from your local machine**

<CommandBlock>
ssh -L 9428:localhost:9428 pixee@<your-hostname>
</CommandBlock>

**Step 2: Set up port forwarding**

In the SSH session, run:

<CommandBlock>
sudo kubectl port-forward svc/pixee-enterprise-server-victoria-logs-single-server 9428:9428 -n kotsadm
</CommandBlock>

**Step 3: Access the logs interface**

Open your browser and navigate to:

<CommandBlock>
http://localhost:9428/vmui/
</CommandBlock>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

**Step 1: Port forward to VictoriaLogs**

<CommandBlock>
kubectl port-forward svc/pixee-enterprise-server-victoria-logs-single-server 9428:9428 -n pixee-enterprise-server
</CommandBlock>

**Step 2: Access the logs interface**

Open your browser and navigate to:

<CommandBlock>
http://localhost:9428/vmui/
</CommandBlock>

{{/if}}

### LogsQL Query Examples

VictoriaLogs uses LogsQL, a powerful query language for searching and filtering logs. Here are some useful queries:

<CommandBlock>
# Search for errors across all logs
error OR ERROR

# Filter logs by pod name
_stream:{pod="pixee-enterprise-server-platform"}

# Search for specific text in platform logs
_stream:{pod=~"pixee-enterprise-server-platform.*"} AND "webhook"

# Find logs with a specific log level
_stream:{pod=~"pixee-enterprise-server-.*"} AND level="ERROR"

# Search within a time range (use the time picker in VMUI)
_stream:{namespace="kotsadm"} AND "analysis"
</CommandBlock>

### VictoriaLogs Resources

- [VictoriaLogs Documentation](https://docs.victoriametrics.com/victorialogs/)
- [LogsQL Query Language](https://docs.victoriametrics.com/victorialogs/logsql/) - Complete LogsQL reference
- [VictoriaLogs VMUI](https://docs.victoriametrics.com/victorialogs/querying/#vmui) - Web UI guide for logs

## Viewing Pods

To view the pods in your deployment, use the namespace you installed Pixee into:

- **Embedded Cluster**: `kotsadm`
- **Helm Deployment**: `pixee-enterprise-server`, `default`, or your selected namespace

### List Running Pods

<CommandBlock>
kubectl get pods -n <namespace> --field-selector status.phase=Running
</CommandBlock>

### View All Pods with Status

<CommandBlock>
kubectl get pods -n <namespace>
</CommandBlock>

## Viewing Logs

### Basic Log Viewing

To view logs for a specific service, use the `kubectl logs` command. For example, to view logs for the platform service:

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

<CommandBlock>
kubectl logs deployment.apps/pixee-enterprise-server-platform -n kotsadm
</CommandBlock>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

<CommandBlock>
kubectl logs deployment.apps/pixee-enterprise-server-platform -n pixee-enterprise-server
</CommandBlock>

{{/if}}

### Viewing Recent Logs

To narrow down to the last 25 lines with timestamps:

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

<CommandBlock>
kubectl logs deployment.apps/pixee-enterprise-server-platform -n kotsadm --tail 25 --timestamps
</CommandBlock>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

<CommandBlock>
kubectl logs deployment.apps/pixee-enterprise-server-platform -n pixee-enterprise-server --tail 25 --timestamps
</CommandBlock>

{{/if}}

### Following Logs in Real-Time

To continuously stream new logs as they are generated, use the `--follow` flag:

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

<CommandBlock>
kubectl logs deployment.apps/pixee-enterprise-server-platform -n kotsadm --follow --tail 25 --timestamps
</CommandBlock>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

<CommandBlock>
kubectl logs deployment.apps/pixee-enterprise-server-platform -n pixee-enterprise-server --follow --tail 25 --timestamps
</CommandBlock>

{{/if}}

## Common Deployments to Monitor

Here are the main Pixee Enterprise Server deployments you may need to view logs for:

| Deployment                         | Purpose               |
| ---------------------------------- | --------------------- |
| `pixee-enterprise-server-platform` | Main platform service |
| `pixee-enterprise-server-analysis` | Analysis service      |

### Example: Viewing Analysis Service Logs

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

<CommandBlock>
kubectl logs deployment.apps/pixee-enterprise-server-analysis -n kotsadm --tail 50 --timestamps
</CommandBlock>

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

<CommandBlock>
kubectl logs deployment.apps/pixee-enterprise-server-analysis -n pixee-enterprise-server --tail 50 --timestamps
</CommandBlock>

{{/if}}

## Viewing Logs for Specific Pods

If you need to view logs for a specific pod (rather than a service), first get the pod name:

<CommandBlock>
kubectl get pods -n <namespace>
</CommandBlock>

Then view the logs:

<CommandBlock>
kubectl logs <pod-name> -n <namespace>
</CommandBlock>

### Multiple Containers in a Pod

If a pod has multiple containers, specify the container name:

<CommandBlock>
kubectl logs <pod-name> -c <container-name> -n <namespace>
</CommandBlock>

## Troubleshooting Common Issues

### No Logs Appearing

If no logs are appearing:

1. Verify the pod is running:

   <CommandBlock>
   kubectl get pods -n <namespace>
   </CommandBlock>

2. Check pod status and events:

   <CommandBlock>
   kubectl describe pod <pod-name> -n <namespace>
   </CommandBlock>

3. Check if the service has active endpoints:
   <CommandBlock>
   kubectl get endpoints -n <namespace>
   </CommandBlock>

### Pod Keeps Restarting

If a pod is repeatedly restarting, check the previous container's logs:

<CommandBlock>
kubectl logs <pod-name> -n <namespace> --previous
</CommandBlock>

## Additional Resources

For more information on the `kubectl logs` command, see the [Kubernetes documentation](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs).
