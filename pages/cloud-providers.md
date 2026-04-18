---
title: Cloud Providers
---

# Cloud Providers

This section provides cloud provider-specific guidance and resource examples for deploying Pixee Enterprise Server on major cloud platforms.

## AWS

Configuration and setup information for deploying Pixee Enterprise Server on Amazon Web Services.

### Notes

When installing in EKS v1.30+, persistent volumes need to have the defaultStorageClass set. This is especially important if using the embedded database or embedded object store. Set the following in your `values.yaml`:

```yaml
global:
  defaultStorageClass: "gp2"
```

### Resources

For Helm deployments on EKS, AWS resources typically include:

- RDS / Aurora PostgreSQL (small) for external database
- 2x S3 buckets for external object storage
- IAM role with S3 permissions (if using IRSA)
- EKS cluster with appropriate node groups
- Application Load Balancer (if using ALB ingress controller)

### Service Account Authentication

For enhanced security when using external object storage, you can configure service account authentication instead of using static AWS credentials. This approach leverages cloud provider IAM roles and eliminates the need for long-lived access keys.

<Note>
IAM Roles for Service Accounts (IRSA) are currently supported with helm installations. Please reach out to Pixee Support if you need assistance with this setup.
</Note>

#### AWS S3 with IRSA (IAM Roles for Service Accounts)

This section covers AWS S3 access from EKS clusters. For other cloud providers accessing their native object stores (GCS, Azure Blob), similar workload identity patterns apply but are not covered in this guide.

##### Prerequisites

1. EKS cluster with OIDC identity provider enabled
2. IAM role with appropriate S3 permissions
3. Trust relationship configured between the IAM role and the EKS service account

##### Setup Steps

1. Create IAM Role and Policy

   Create an IAM policy with the required S3 permissions:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": ["s3:ListBucket"],
         "Resource": ["arn:aws:s3:::pixee-analysis-input"]
       },
       {
         "Effect": "Allow",
         "Action": [
           "s3:GetObject",
           "s3:PutObject",
           "s3:DeleteObject",
           "s3:GetObjectVersion"
         ],
         "Resource": ["arn:aws:s3:::pixee-analysis-input/*"]
       }
     ]
   }
   ```

2. Create Kubernetes Service Account

   Create a service account with the IAM role annotation:

   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: pixee-s3-service-account
     namespace: pixee-enterprise-server
     annotations:
       eks.amazonaws.com/role-arn: "arn:aws:iam::123456789012:role/pixee-s3-role"
   ```

3. Configure Helm Values

   Set the following in your `values.yaml`:

   ```yaml
   global:
     pixee:
       serviceAccount:
         create: false
         name: "pixee-s3-service-account"
       objectStore:
         embedded: false
         endpoint: "https://s3.amazonaws.com"
         region: "us-east-1"
         credentialType: "default" # Use IRSA
         # username and password are not required with IRSA
   ```

### External RDS Database Configuration

If using an external database such as Amazon RDS for PostgreSQL you can reference an existing Kubernetes secret instead of passing the password directly through helm values.

1. See the [installation prerequisites](installation/prerequisites.md#database) for database requirements.

2. Create a Kubernetes secret with a `password` key that contains the password for the PostgreSQL user to be used by Pixee.

3. Configure Helm Values

   ```yaml
   database:
     embedded: false
     host: <RDS ENDPOINT>
     port: <RDS PORT>
     name: "pixee_platform"
     username: "pixee"
     existingSecret: <EXISTING SECRET NAME>
   ```

## Azure

Configuration and setup information for deploying Pixee Enterprise Server on Microsoft Azure.

### Resources

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For Embedded Cluster deployments on Azure VMs, resources typically include:

- Resource Group (if it doesn't already exist)
- SSH Key (stored in Azure; used by the VM)
- Virtual Network (VNet)
- Subnet (within the VNet)
- Network Security Group (NSG)
- Inbound rule for TCP on ports: 30000, 443, and 22, 80 temporarily
- Public IP Address (Standard, static)
- Network Interface (NIC) (linked to VNet, subnet, NSG, and the public IP)
- Optional: Azure DNS Zone (if you provide a domain)
- DNS A record pointing to the public IP
- Virtual Machine (image: Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest, attached to the resources above)
- Size: Standard_D8s_v3 w/ 512 GB, Premium_LRS os disk
- Azure Cognitive Services (OpenAI) resource
  - OpenAI Model Deployment ("o3-mini")

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments on AKS, Azure resources typically include:

- Resource Group (if it doesn't already exist)
- Virtual Network (VNet)
- Subnet (within the VNet)
- Network Security Group (NSG)
- Inbound rule for TCP on ports: 443
- Public IP Address (Standard, static)
- Optional: Azure DNS Zone (if you provide a domain)
- DNS A record pointing to the public IP
- Kubernetes cluster (AKS) with worker nodes sized appropriately
  - Node size equivalent to Standard_D8s_v3 or better
- Azure Cognitive Services (OpenAI) resource
  - OpenAI Model Deployment ("o3-mini")

{{/if}}

## Google Cloud Platform

Configuration and setup information for deploying Pixee Enterprise Server on Google Cloud Platform.

### Notes

You can utilize the built-in ingress controller for Google Kubernetes Engine by setting the following in `values.yaml`:

```yaml
global:
platform:
  service:
    type: ClusterIP
  ingress:
    enabled: true
    className: "gce"
    annotations:
      "kubernetes.io/ingress.class": "gce"
    hosts:
      - host: ""
        paths:
          - path: "/"
            pathType: "Prefix"
```

### Resources

For Helm deployments on GKE, Google Cloud resources typically include:

- Google Kubernetes Engine
- Cloud SQL

## Oracle Cloud Infrastructure

Configuration and setup information for deploying Pixee Enterprise Server on Oracle Cloud Infrastructure.

### Resources

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For Embedded Cluster deployments on OCI VMs, resources typically include:

- Virtual Cloud Network (VCN)
- Subnet (within the VCN)
- Network Security Group (NSG)
- Security List or NSG Rules (allowing ingress on ports 30000, 443, 22 (temp), 80 (temp))
- Reserved Public IP (if applicable)
- Virtual Network Interface Card (VNIC) (attached to the instance, associated with VCN, subnet, NSG, and Public IP)
- OCI DNS Zone (if managing the domain in OCI)
- DNS A Record (pointing to the Reserved Public IP in OCI DNS)
- Compute Instance (Ubuntu 22.04 image from OCI Marketplace or Platform Images)
  - VM.Standard3.Flex (8 OCPUs, 64GB RAM) with a 512 GB Block Volume (NVMe or Balanced option)
- SSH Key Pair
- OCI Generative AI (if available) or Custom Model Deployment in OCI Data Science
- OCI Generative AI Service Deployment (if applicable) or OCI AI Services (custom model in Data Science or AI Text Services)

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments on OKE, OCI resources typically include:

- Virtual Cloud Network (VCN)
- Subnet (within the VCN)
- Network Security Group (NSG)
- Security List or NSG Rules (allowing ingress on ports 443)
- Reserved Public IP (if applicable)
- OCI DNS Zone (if managing the domain in OCI)
- DNS A Record (pointing to the Reserved Public IP in OCI DNS)
- Kubernetes cluster (OKE) with worker nodes sized appropriately
  - Node size equivalent to VM.Standard3.Flex (8 OCPUs, 64GB RAM)
- OCI Generative AI (if available) or Custom Model Deployment in OCI Data Science
- OCI Generative AI Service Deployment (if applicable) or OCI AI Services (custom model in Data Science or AI Text Services)

{{/if}}
