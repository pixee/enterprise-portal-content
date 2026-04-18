---
title: Basic Configuration
---

# Basic Configuration

After installation, you'll need to configure the basic settings for Pixee Enterprise Server. These settings include the domain name, protocol, and ingress configuration.

## Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Configuration is done through the admin console configuration page available after installation at:

<CodeBlock>
https://<domain name or ip address>:30000
</CodeBlock>

When you load the admin console page you will be prompted to enter your admin password. The first time configuring after installation you will be directed through a workflow that will step you through configuring Pixee Enterprise Server.

### Basic Settings

The settings under the `Basic Settings` section all require your input. Here you will find the required settings for the Pixee Enterprise Server domain name, TLS options, and AI model provider settings. Make sure you review all of these settings for completeness and accuracy.

#### Domain

Enter the domain name you have assigned to your Pixee Enterprise Server. If you have not assigned a domain name you can enter the public IP address of your Pixee Enterprise server instead but this will limit your TLS options.

#### Protocol

Select the protocol that will be used for your Pixee Enterprise Server. If you select `HTTP`, ingress traffic to your Pixee Enterprise Server will be un-encrypted. The `HTTP` option is for quick testing configurations or when you have an external system like an App Gateway or Load Balancer that is terminating TLS instead of Pixee Enterprise Server. If you have an external system (i.e. App Gateway, Load Balancer, etc.) acting as a reverse proxy to your Pixee Enterprise Server, make sure you provide reverse proxy settings in the Advanced Settings section. If you select `HTTPS`, ingress traffic to your Pixee Enterprise Server will be encrypted, and you will be prompted to configure TLS.

#### Authentication

Pixee Enterprise Server currently supports the following OIDC providers:

- [Embedded Provider](authentication.md#embedded-oidc-provider)
- [Google](authentication.md#google-authentication)
- [Microsoft Entra](authentication.md#microsoft-entra-authentication)
- [Okta](authentication.md#okta-authentication)

Select the OIDC provider you want to use for authentication. If you select `Embedded Provider`, Pixee Enterprise Server will use its built-in OIDC provider. For other providers, you will need to provide the necessary configuration details such as client ID, client secret, and issuer URL.

See [Authentication](authentication.md) for more information on specific provider configuration.

#### AI Providers

To use OpenAI directly, select OpenAI and enter your OpenAI API key.
To use Azure OpenAI, select Azure OpenAI and enter your Azure OpenAI resource endpoint, key, and model deployment names for `o3-mini`.

For Databricks please see: [Databricks AI Serving Endpoints](../ai-providers.md#databricks-ai).

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

Create a `values.yaml` file and configure the following basic settings:

### Domain

Set the URL where your Pixee Enterprise Server will be accessible (if no domain name is available, use an external IP address):

<CodeBlock language="yaml">
global:
  pixee:
    domain: "<your pixee enterprise server domain name>"
</CodeBlock>

### Protocol

Set the HTTP protocol (`http` or `https`) used to access your Pixee Enterprise Server:

<CodeBlock language="yaml">
global:
  pixee:
    protocol: "https"
</CodeBlock>

<Note>
If you are using TLS to secure traffic to your Pixee Enterprise Server set this to `https`, even if you terminate TLS outside your cluster.
</Note>

### Ingress

If you are using an ingress controller, you can enable and configure the Pixee Enterprise Server ingress resource as follows:

<CodeBlock language="yaml">
platform:
  proxy:
    # enable proxy configuration with ingress to allow headers from the ingress controller
    enabled: true
  ingress:
    enabled: true
    className: "<your ingress controller class name (i.e. nginx, gce, etc)"
    hosts:
      - host: "<your pixee enterprise server domain name>"
        paths:
          - path: "/"
            pathType: "Prefix"
    # If you are securing your Pixee Enterprise Server with TLS via ingress, set the following
    tls:
      - hosts:
          - "<your pixee enterprise server domain name>"
        secretName: "<your tls certificate secret name>"
</CodeBlock>

### AI Model Provider - OpenAI

To configure access to the OpenAI API, set the following:

<CodeBlock language="yaml">
global:
  pixee:
    ai:
      openai:
        key: "<your OpenAI API key>"
        # Optional: Custom OpenAI API base URL (e.g., for Azure OpenAI compatible endpoints)
        # baseUrl: "https://example.cloud.databricks.com/serving-endpoints"
        # -- Use an existing secret instead of creating one
        existingSecret: ""
        secretKeys:
          # -- Secret key containing the api key
          apiKey: "key"
</CodeBlock>

### AI Model Provider - Azure OpenAI

To configure access to Azure OpenAI, set the following:

<CodeBlock language="yaml">
global:
  pixee:
    ai:
      azure:
        enabled: true
        key: "<your Azure OpenAI key>"
        # -- Use an existing secret instead of creating one
        existingSecret: ""
        secretKeys:
          # -- Secret key containing the api key
          apiKey: "key"
        endpoint: "<your Azure OpenAI endpoint>"
        deployments:
          o3-mini: "<your model deployment name for o3-mini>"
</CodeBlock>

### Databricks AI Serving Endpoints

Pixee Enterprise Server can integrate with Databricks AI. See [Databricks AI](../ai-providers.md#databricks-ai) for more information.

To configure Databricks AI serving endpoints, set the following:

<CodeBlock language="yaml">
global:
  pixee:
    ai:
      openai:
        enabled: true
        key: "<your Databricks PAT or API key>"
        baseUrl: "https://<your-databricks-workspace>.cloud.databricks.com/serving-endpoints"
        # -- Use an existing secret instead of creating one
        existingSecret: ""
        secretKeys:
          # -- Secret key containing the api key
          apiKey: "key"
</CodeBlock>

<Note>
The baseUrl should point to your Databricks workspace serving endpoints. Ensure the required model endpoint (`o3-mini`) is deployed and accessible.
</Note>

{{/if}}
