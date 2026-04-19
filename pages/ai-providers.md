---
title: AI Providers Overview
---

# AI Providers Overview

This section provides AI provider-specific guidance and resource examples for deploying Pixee Enterprise Server on major cloud platforms.

## Advanced AI Model Configuration

### SCA Models

Software Composition Analysis (SCA) models enable enhanced vulnerability detection in dependencies and third-party libraries. This feature is disabled by default and can be enabled in the AI settings.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure SCA models in Embedded Cluster deployments:

Navigate to the admin console, select the `Config` tab, then go to the `AI Settings` section.

- **Enable SCA Models**: Toggle to enable Software Composition Analysis models (disabled by default)
- **SCA Model Name**: Optionally specify a custom LLM model for SCA analysis (only available when SCA is enabled, defaults to gpt-4.1)

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure SCA models in Helm deployments:

<CommandBlock>
global:
  pixee:
    ai:
      # Enable SCA models (disabled by default)
      scaModelsEnabled: true
      # Optionally specify a custom model name (defaults to gpt-4.1)
      scaModelName: "gpt-4.1"
</CommandBlock>

{{/if}}

### Deep Research Models

Deep Research models provide advanced analysis capabilities for complex code understanding tasks. These models are only used when SCA is enabled.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure Deep Research models in Embedded Cluster deployments:

Navigate to the admin console, select the `Config` tab, then go to the `AI Settings` section.

- **Deep Research Model Name**: Optionally specify a custom LLM model for deep research analysis (only available when SCA is enabled, defaults to o4-mini-deep-research)

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure Deep Research models in Helm deployments:

<CommandBlock>
global:
  pixee:
    ai:
      # Enable SCA first (required for deep research models)
      scaModelsEnabled: true
      # Then specify a custom model for deep research (defaults to o4-mini-deep-research)
      deepResearchModelName: "o4-mini-deep-research"
</CommandBlock>

{{/if}}

## OpenAI

### Requirements

AI Provider integration requires:

- An AI provider API key with access to the required models
- An AI provider endpoint, or default to the provider's public API endpoint
- Model names for the reasoning and fast models from your chosen AI provider

For OpenAI, see OpenAI's page on creating an [API Key](https://help.openai.com/en/articles/4936850-where-do-i-find-my-openai-api-key) for more information.

## OpenAI-compatible APIs

Providers such as Azure AI Foundry and AWS Bedrock provide endpoints for connecting to both OpenAI and non-OpenAI models via a consistent OpenAI-compatible API. This enables connection to models from these providers with the same configuration as if it was OpenAI directly.

### Requirements

- An AI provider endpoint
- Model names for the reasoning and fast models from your chosen AI provider
- An AI provider API key with access to the required models

### Example: Connecting to DeepSeek-R1 via Azure AI Foundry

Azure AI Foundry hosts a variety of foundation models from industry-leading providers. In this example, Pixee will be configured to connect to DeepSeek-R1 using the OpenAI-compatible API endpoints using the same process for connecting to OpenAI's API directly. This example assumes that the model has already been deployed in the target Azure environment, and a model deployment API key has already been created.

#### Input the OpenAI-compatible endpoint for your deployment

Azure AI Foundry provides a cononacle URL endpoint for AI Foundry model deployments. It's usually of the format `https://{foundary-instance-name}-resource.service.ai.azure.com/models`.

#### Input your API key

Copy the deployment's endpoint key from the Azure AI Foundry portal and paste it into the API key field.

#### Specify the reasoning and fast model names

Find the `DeepSeek-R1` model name in the AI Foundry portal and paste it into the reasoning and fast model name fields. We want to use the same model for this example to keep things simple. But we could choose a different model if we wanted to, so long as it's available in our AI Foundry instance.

[![dsr1-openai-model-names.png]({{asset "assets/ai-providers/dsr1-openai-model-names.png"}})]({{asset "assets/ai-providers/dsr1-openai-model-names.png"}})

#### Verify with preflight checks

Preflight checks should run after the configuration is saved and should successfully connect to the model.

[![dsr1-openai-preflight.png]({{asset "assets/ai-providers/dsr1-openai-preflight.png"}})]({{asset "assets/ai-providers/dsr1-openai-preflight.png"}})

## Azure OpenAI

### Requirements

Azure OpenAI integration requires:

- Integration with [Azure OpenAI](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/create-resource?pivots=web-portal) requires model deployments of the following recommended models:
  - o3-mini (version: 2025-01-31) -- as fast model
  - o3-mini (version: 2025-01-31) -- as reasoning model

## Databricks AI

### Requirements

Databricks AI integration requires:

- Integration with Databricks AI serving endpoints requires:
  - Databricks workspace with Mosaic AI Model Serving enabled
  - External model endpoints for "o3-mini" as fast and reasoning models
  - Databricks personal access token with serving endpoint access

### Configuration

To integrate Pixee Enterprise Server with Databricks AI serving endpoints:

1. **Create the required serving endpoints in Databricks** following the [Databricks external models documentation](https://docs.databricks.com/aws/en/generative-ai/external-models/). You need to deploy the following endpoint:
   - `o3-mini` (for o3-mini model)

2. **Configure Pixee Enterprise Server** to use your Databricks workspace URL as the OpenAI base URL with your Databricks PAT as the API key.

3. **Verify connectivity** by checking that both endpoints are accessible from your Pixee Enterprise Server deployment.

<Note>
Databricks integration uses the OpenAI-compatible API, so select "OpenAI" as the provider type when configuring through the admin console, and be sure to update the OpenAI Endpoint to match your Databricks base url.
</Note>

### Common Issues

- Ensure your Databricks PAT has permission to access the serving endpoints
- Verify network connectivity between Pixee Enterprise Server and your Databricks workspace
- Check that the required endpoint ("o3-mini") is deployed and running
- When using the embedded cluster installation, select "OpenAI" as the provider type since Databricks uses OpenAI-compatible APIs
- Be sure to set the OpenAI endpoint to your Databricks base url

## Azure Anthropic

Azure Anthropic allows you to access Anthropic models (such as Claude) through Azure's infrastructure.

### Requirements

- An Azure Anthropic API endpoint for your model deployments
- An API key with access to the required models
- Model names for the reasoning and fast models

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `AI Settings` section.

Select **Azure Anthropic** as the Default LLM Provider and configure:

- **API Key**: Your Azure Anthropic API key
- **Endpoint**: The Azure Anthropic API endpoint for your model deployments
- **Reasoning Model Name**: Model for complex reasoning tasks (default: claude-sonnet-4-20250514)
- **Fast Model Name**: Model for quick response tasks (default: claude-sonnet-4-20250514)

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure Azure Anthropic in Helm deployments, add the following to your `values.yaml`:

<CommandBlock>
global:
  pixee:
    ai:
      enabled: true
      default:
        provider: "azure-anthropic"
        model: "claude-sonnet-4-20250514"
        apiKey: "<your Azure Anthropic API key>"
        endpoint: "<your Azure Anthropic endpoint>"
      reasoning:
        model: "claude-sonnet-4-20250514"
      fast:
        model: "claude-sonnet-4-20250514"
</CommandBlock>

{{/if}}

## Web Search LLM Model

When an AI provider is configured, you can optionally specify a model for web-search-enabled queries. This model is used for tasks that benefit from real-time web search capabilities during analysis.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `AI Settings` section.

When using OpenAI or Azure AI Foundry as your provider, a **Web Search Model Name** field is available. The default is `gpt-5.2`.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure a web search model in Helm deployments:

<CommandBlock>
global:
  pixee:
    ai:
      webSearch:
        model: "gpt-5.2"
</CommandBlock>

<Note>
Web search models are currently supported for OpenAI and Azure AI Foundry providers only.
</Note>

{{/if}}

## Oracle Cloud Infrastructure Generative AI Services

### Requirements

Oracle Cloud Infrastructure (OCI) integration requires:

- Integration with Oracle Cloud Infrastructure Generative AI Services
  - Llama and custom models can be deployed, please contact support for up-to-date instructions based on your deployment type.
  -
