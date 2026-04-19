---
title: Prerequisites
---

# Prerequisites

Before installing Pixee Enterprise Server, you'll need to provision the necessary infrastructure.

## Common Requirements

### Database

For trial installations you can use the embedded database and skip this section. For production environments, we recommend creating an external database:

- PostgreSQL 17.4+
- 10Gb+ available disk space
- Network connectivity between Pixee Enterprise Server Kubernetes cluster and database
- Create database named `pixee_platform` (or any name you choose)
- Create user with permissions to `pixee_platform` database

### External Object Store

You can configure Pixee Enterprise Server to use an external object store. If you prefer to use the in-cluster, embedded object store, you may skip this section.

### Requirements

The following are requirements of an external object store compatible with Pixee Enterprise Server:

- The object store and the Kubernetes cluster are able to communicate over the network
- The object store exposes a S3 compatible API
- A bucket has been created for use as the `pixee-analysis-input` bucket

### AI Providers

Advanced AI capabilities are required. Please review the [AI Providers](../ai-providers.md) documentation for more information for supported AI providers in Pixee Enterprise Server.

## Infrastructure Requirements

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Create a VM with the following specifications:

- Linux distro: Ubuntu 24.04+ (recommended) or Enterprise Linux 9 (RHEL 9, Rocky Linux, AlmaLinux)
- `systemd` installed
- For Enterprise Linux 9: SELinux must be in **permissive** mode (enforcing mode is not supported)
- Allows traffic to egress to the internet (more details [here](../troubleshooting.md#what-internet-access-is-necessary-for-pixee-enterprise-server))
- Allows HTTPS traffic to ingress to port 443
- Allows HTTPS traffic to ingress to port 30000 (for embedded cluster admin console)
- 8 vCPU, 32 GB RAM
- 100GB+ disk with <10ms write latency (i.e. SSD/NVME)

**DNS Configuration**: Create the appropriate DNS records so that a domain name resolves to your provisioned virtual machine.

**TLS Certificate**: To encrypt traffic to Pixee Enterprise Server, you'll need to generate/acquire a TLS certificate for use with your selected domain name. With the Embedded Cluster installation method, Pixee Enterprise Server can automate the TLS certificate request using LetsEncrypt or self-signed certificates as part of the configuration process.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

Select or create a Kubernetes cluster with the following available to Pixee Enterprise Server:

- 8+ vCPU
- 32+ GB RAM
- 100GB+ disk with <10ms write latency (i.e. SSD/NVME)
- Allow outgoing HTTP traffic to the internet (more details [here](../troubleshooting.md#what-internet-access-is-necessary-for-pixee-enterprise-server))
- Allow incoming HTTPS traffic to port 443
- (optional) ingress controller installed and configured

**DNS Configuration**: Create the appropriate DNS records so that a domain name resolves to your Kubernetes cluster.

**TLS Certificate**: To encrypt traffic to Pixee Enterprise Server, you'll need to generate/acquire a TLS certificate for use with your selected domain name. With Helm, you have multiple options for TLS certificate management: using [cert-manager](https://cert-manager.io/docs/installation/) to automatically provision TLS certificates, using pre-existing TLS certificates as Kubernetes secrets, or terminating TLS outside the cluster (i.e., via a load balancer).

**Additional Helm Requirements**:

- Kubectl installed and configured to access the target cluster
- Helm CLI installed (version >3.15)
- `preflight` and `support-bundle` plugins from [troubleshoot.sh](https://troubleshoot.sh/docs/#installation) installed in the target cluster
- Access to image registry (images.pixee.ai) from Kubernetes cluster

<Tip>
If you need to pull images from an internal registry, you will need to update your `values.yaml` to override the registry and pullSecrets values for all images listed in the reference section identified by the patterns `**.image.registry` and `**.image.pullSecrets`. In addition, if your registry requires authentication you will need to create a `dockerconfigjson` type secret to authenticate with your internal registry in the pullSecret value for each image.
</Tip>

{{/if}}
