---
title: Update Instructions
---

# Update Instructions

Instructions for updating Pixee Enterprise Server to newer versions.

## Update Process

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To update Pixee Enterprise Server in Embedded Cluster deployments follow:

Updating Pixee Enterprise Server is a simple process:

1. Visit the admin console in your browser: `https://<domain name or vm ip>:30000`
2. Login with the admin console password set during installation
3. New available versions will be visible in the `Available Updates` list under the `Version History` tab
4. Click `deploy` next to the desired version you want to upgrade to
5. Confirm configuration
6. Verify preflight checks
7. Confirm deployment

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To update Pixee Enterprise Server in Helm Deployment follow:

To update a Pixee Enterprise Server release, use the helm upgrade command. This is good for when you need to apply template changes to the running release or update configuration values.

To change variable values in a release, use the `helm upgrade` command with the `-f` flag and provide your updated values.yaml file:

```shell
helm upgrade pixee-enterprise-server oci://registry.pixee.ai/pixee/stable/pixee-enterprise-server -f values.yaml -n pixee-enterprise-server
```

{{/if}}
