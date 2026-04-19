---
title: Verify Installation
---

# Verify Installation

After installation is complete, you can verify your installation with the following steps.

## Health Check Endpoint

Both deployment methods provide the same health check endpoint to verify the Pixee Enterprise service status:

<CommandBlock>
curl https://<domain or ip>/q/health
</CommandBlock>

Expected response:

<CommandBlock>
{
  "status": "UP",
  "checks": [
    {
      "name": "SmallRye Reactive Messaging - liveness check",
      "status": "UP"
    },
    {
      "name": "Pixee Server health check",
      "status": "UP",
      "data": {
        "server-version": "2024-11-03-653a81d"
      }
    },
    {
      "name": "Database connections health check",
      "status": "UP",
      "data": {
        "<default>": "UP"
      }
    },
    {
      "name": "SmallRye Reactive Messaging - readiness check",
      "status": "UP"
    },
    {
      "name": "SmallRye Reactive Messaging - startup check",
      "status": "UP"
    }
  ]
}
</CommandBlock>

## Kubernetes Resources

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To verify Kubernetes resources in Embedded Cluster deployments follow:

1. Open a terminal session on the VM and run the following commands:

   <CommandBlock>
   sudo ./pixee shell
   kubectl get all -n kotsadm
   </CommandBlock>

2. Verify the Pixee Enterprise Server is ready by viewing pods and services, making sure all are in the `ready` state.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To verify Kubernetes resources in Helm Deployment follow:

Verify the application is properly deployed by viewing pods and services, making sure all are in the `ready` state:

<CommandBlock>
kubectl get all -n pixee-enterprise-server
</CommandBlock>

{{/if}}

## GitHub App Connectivity

If you enabled GitHub integration and created a custom GitHub app, you can verify your GitHub App connectivity by checking your GitHub App's event log.

This log can be accessed through your GitHub App's settings under the "Advanced" section. See [GitHub.com](https://docs.github.com/en/webhooks/testing-and-troubleshooting-webhooks/viewing-webhook-deliveries#viewing-deliveries-for-github-app-webhooks) for more information.
