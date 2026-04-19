---
title: Security Integrations
---

# Security Integrations

This section covers integrating Pixee Enterprise Server with various security scanning and analysis tools.

## HCL AppScan Integration

HCL AppScan integration allows Pixee Enterprise Server to communicate with your existing AppScan security scans to analyze, react to, fix and update issues.

### Requirements

HCL AppScan integration requires:

- An AppScan Base URI (defaults to https://cloud.appscan.com)
- An AppScan Key ID and Key Secret
  - The key ID must be attached to a role with permissions to post comments on issues
- Webhook authentication credentials:
  - **Basic Auth** (recommended): Username and password for HTTP Basic authentication on incoming webhooks.
  - **Webhook Secret** (deprecated): The secret is embedded in the webhook URL path. This method is deprecated in favor of Basic Auth.

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Security Tool` section.

Select the `AppScan` checkbox to enable HCL AppScan integration.

Enter the following information in the configuration fields:

- **Base URI**: Your AppScan base URI (defaults to https://cloud.appscan.com)
- **Key ID**: Your AppScan key ID with comment permissions
- **Key Secret**: Your AppScan key secret
- **Webhook Authentication Mode**: Choose your authentication method:
  - **Basic Auth (Username/Password)** (recommended): Enter username and password for HTTP Basic authentication on incoming webhooks.
  - **Webhook Secret** (deprecated): Enter a shared secret that will be embedded in the webhook URL. This method is deprecated in favor of Basic Auth.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CommandBlock>
platform:
  pixeebot:
    appscan:
      apiKeyId: "your-appscan-key-id"
      apiKeySecret: "your-appscan-key-secret"
      webhook:
        user: "your-webhook-username"
        password: "your-webhook-password"
      # Use existing secret instead of creating one
      existingSecret: ""
      secretKeys:
        apiKeySecretKey: "apiKeySecret"
        webhookUserKey: "webhookUser"
        webhookPasswordKey: "webhookPassword"
</CommandBlock>

{{/if}}

### Webhook Configuration

To receive notifications from AppScan, you'll need to configure two webhooks in your AppScan presence server using Basic Auth.

#### Creating the Authorization Header

First, generate the Base64-encoded authorization header using the webhook username and password you configured in Pixee Enterprise Server:

<CommandBlock>
echo -n "username:password" | base64
</CommandBlock>

This will output a Base64 string like `dXNlcm5hbWU6cGFzc3dvcmQ=`. Prepend `Basic ` to create the full authorization header value.

#### Webhook 1: Scan Execution Completed

This webhook notifies Pixee Enterprise Server when an AppScan scan completes. Use the AppScan Webhook API to create it with the following request body:

<CommandBlock>
{
  "AuthorizationHeader": "Basic <your-base64-encoded-credentials>",
  "PresenceId": "<your-presence-id>",
  "Uri": "https://<your-pixee-server>/api/v1/integrations/appscan-default/webhooks/_/ScanExecutionCompleted/{SubjectId}",
  "Global": true,
  "AssetGroupId": "<your-asset-group-id>",
  "Event": "ScanExecutionCompleted"
}
</CommandBlock>

#### Webhook 2: New Patch Request

This webhook notifies Pixee Enterprise Server when a new patch is requested in AppScan. Use the AppScan Webhook API to create it with the following request body:

<CommandBlock>
{
  "AuthorizationHeader": "Basic <your-base64-encoded-credentials>",
  "PresenceId": "<your-presence-id>",
  "Uri": "https://<your-pixee-server>/api/v1/integrations/appscan-default/webhooks/CreatePatch",
  "Global": true,
  "AssetGroupId": "<your-asset-group-id>",
  "Event": "NewPatchRequest",
  "RequestMethod": "POST",
  "RequestBody": "{\"patch_id\": \"{SubjectId}\"}",
  "ContentType": "application/json"
}
</CommandBlock>

#### Placeholder Reference

Replace the following placeholders in both webhooks:

- `<your-base64-encoded-credentials>`: The Base64-encoded `username:password` string from the command above
- `<your-presence-id>`: Your AppScan presence server ID
- `<your-pixee-server>`: Your Pixee Enterprise Server hostname
- `<your-asset-group-id>`: Your AppScan asset group ID

For detailed instructions on configuring AppScan webhooks, refer to the [AppScan Webhook API Documentation](https://cloud.appscan.com/swagger/index.html#/Webhooks/Webhooks_Create){:target="\_blank"}.

## Arnica Integration

Arnica integration allows Pixee Enterprise Server to communicate with your existing Arnica security platform to analyze, react to, fix and update issues.

### Requirements

Arnica integration requires:

- An Arnica API key

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Security Tool` section.

Select the `Arnica` checkbox to enable Arnica integration.

Enter the following information in the configuration fields:

- **API Key**: Your Arnica API key

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CommandBlock>
platform:
  arnica:
    apiKey: "your-arnica-api-key"
    # Use existing secret instead of creating one
    existingSecret: ""
    secretKeys:
      # -- The secret key containing the apiKey
      apiKeyKey: "apiKey"
</CommandBlock>

{{/if}}

## Black Duck Integration

Black Duck integration allows Pixee Enterprise Server to communicate with your existing Black Duck security scans to analyze, react to, fix and update issues.

### Requirements

Black Duck integration requires:

- A Black Duck access token

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Security Tool` section.

Select the `Black Duck` checkbox to enable Black Duck integration.

Enter the following information in the configuration fields:

- Access Token: Your Black Duck access token

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CommandBlock>
platform:
  blackduck:
    accessToken: "your-blackduck-access-token"
    # Use existing secret instead of creating one
    existingSecret: ""
    secretKeys:
      # -- The secret key containing the accessToken
      accessTokenKey: "accessToken"
</CommandBlock>

{{/if}}

## SonarQube Integration

SonarQube integration allows Pixee Enterprise Server to communicate with your existing SonarQube security scans to analyze, react to, fix and update issues.

Pixee Enterprise Server can integrate with both SonarQube Cloud and SonarQube Server.

### Requirements

SonarQube integration requires:

- A SonarQube personal access token with access to retrieve issues and hotspots for the projects that will be integrated with Pixee Enterprise Server
- A [webhook](https://docs.sonarsource.com/sonarqube-cloud/advanced-setup/webhooks/){:target="\_blank"} secret for receiving scan notifications. When creating the webhook, set the URL to `https://<domain>/api/v1/integrations/sonar-default/webhooks`.

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Security Tool` section.

Select the `SonarQube` checkbox to enable SonarQube integration. If you host your own SonarQube server instance, select SonarQube server and enter your SonarQube Server base URI and SonarQube GitHub app if applicable.

For all SonarQube integration types (Server or Cloud) enter the following information in the configuration fields:

- Personal Access Token: Your SonarQube personal access token
- Webhook Secret: Secret for webhook authentication

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CommandBlock>
platform:
  sonar:
    # For SonarQube Server integration, provide your SonarQube server baseUri:
    # baseUri: "https://sonarqube.your-company.com"
    # If you have a custom Sonar GitHub app, provide the GitHub app name:
    # gitHubAppName: "your-sonarqube-github-app-name"
    token: "your-sonarqube-personal-access-token"
    webhookSecret: "your-sonarqube-webhook-secret"
    # -- Use an existing secret instead of creating one
    existingSecret: ""
    secretKeys:
      # -- The secret key containing the token
      tokenKey: "token"
      # -- The secret key containing the webhookSecret
      webhookSecretKey: "webhookSecret"
</CommandBlock>

{{/if}}

### Advanced Filtering Options

SonarQube integration supports advanced filtering to control which findings are retrieved and processed.

#### Software Quality Filtering

Control which types of findings to retrieve:

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

In the admin console `Security Tool` section, use the following checkboxes:

- **Exclude Maintainability Findings**: Select to exclude maintainability findings (code smells), retrieving only security-related issues
- **Exclude Reliability Findings**: Select to exclude reliability findings (bugs), retrieving only security-related issues

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

<CommandBlock>
platform:
  sonar:
    # Exclude maintainability findings (code smells)
    excludeMaintainabilityFindings: true
    # Exclude reliability findings (bugs)
    excludeReliabilityFindings: true
</CommandBlock>

{{/if}}

#### CWE Filtering

Filter findings by specific Common Weakness Enumeration (CWE) identifiers:

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

In the admin console `Security Tool` section:

- **CWE IDs**: Enter a comma-separated list of CWE IDs to filter findings (e.g., `79,89,502,918`). No spaces. When set, this overrides "Filter CWE Top 25" and "Additional CWE IDs".
- **Filter CWE Top 25** _(Deprecated)_: Select to retrieve only findings from the SANS CWE Top 25 list. Ignored when "CWE IDs" is set.
- **Additional CWE IDs** _(Deprecated)_: Enter comma-separated CWE IDs to include (e.g., `611,918,1234`). No spaces. Ignored when "CWE IDs" is set.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

<CommandBlock>
platform:
  sonar:
    # Explicit CWE ID list (overrides filterCweTop25 and additionalCweIds)
    cweIds: "79,89,502,918"
    # Deprecated - use cweIds instead
    # filterCweTop25: true
    # additionalCweIds: "611,918,1234"
</CommandBlock>

{{/if}}

#### Example Configurations

**Custom CWE list (recommended)**:

<CommandBlock>
platform:
  sonar:
    cweIds: "79,89,502,918"
    excludeMaintainabilityFindings: true
    excludeReliabilityFindings: true
</CommandBlock>

**Security + Reliability (no code smells)**:

<CommandBlock>
platform:
  sonar:
    excludeMaintainabilityFindings: true
</CommandBlock>

**Legacy: SANS Top 25 only** _(deprecated)_:

<CommandBlock>
platform:
  sonar:
    filterCweTop25: true
    excludeMaintainabilityFindings: true
    excludeReliabilityFindings: true
</CommandBlock>

## Veracode Integration

### Requirements

Veracode integration requires:

- A Veracode Key ID and Key Secret

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Security Tool` section.

Select the `Veracode` checkbox to enable Veracode integration.

Enter the following information in the configuration fields:

- Key ID: Your Veracode key ID
- Key Secret: Your Veracode key secret

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CommandBlock>
platform:
  veracode:
    apiKeyId: "your-veracode-key-id"
    apiKeySecret: "your-veracode-key-secret"
    # Use existing secret instead of creating one
    existingSecret: ""
    secretKeys:
      # -- The secret key containing the apiKeySecret
      apiKeySecretKey: "apiKeySecret"
</CommandBlock>

{{/if}}

## Checkmarx Integration

Checkmarx integration allows Pixee Enterprise Server to communicate with your existing Checkmarx One platform to analyze, react to, fix and update security vulnerabilities found in SAST scans.

### Requirements

Checkmarx integration requires:

- A Checkmarx tenant account name. You can find this by going to your Checkmarx One platform and navigating to the `Settings` > `Identity and Access Management` section. The tenant account name appears above the GUID that is your tenant ID. Be sure to use the account name, not the GUID.
- API key with access to retrieve scan results and projects
- Knowledge of your Checkmarx region (US, US2, EU, EU2, DEU, ANZ, IND, SNG, or MEA)

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Security Tool` section.

Select the `Checkmarx` checkbox to enable Checkmarx integration.

Enter the following information in the configuration fields:

- Region: Your Checkmarx region (defaults to US)
- Tenant Account Name: Your Checkmarx tenant account name
- API Key: Your Checkmarx API key

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CommandBlock>
platform:
  checkmarx:
    region: "US" # Available regions: US, US2, EU, EU2, DEU, ANZ, IND, SNG, MEA
    tenantAccountName: "your-checkmarx-tenant-account-name"
    apiKey: "your-checkmarx-api-key"
</CommandBlock>

{{/if}}

### Supported Regions

Checkmarx operates in multiple regions worldwide. The following regions are supported:

- **US**: Default US environment (`https://ast.checkmarx.net`)
- **US2**: Second US environment (`https://us.ast.checkmarx.net`)
- **EU**: European environment (`https://eu.ast.checkmarx.net`)
- **EU2**: Second European environment (`https://eu-2.ast.checkmarx.net`)
- **DEU**: Germany environment (`https://deu.ast.checkmarx.net`)
- **ANZ**: Australia & New Zealand environment (`https://anz.ast.checkmarx.net`)
- **IND**: India environment (`https://ind.ast.checkmarx.net`)
- **SNG**: Singapore environment (`https://sng.ast.checkmarx.net`)
- **MEA**: UAE/Middle East environment (`https://mea.ast.checkmarx.net`)

Make sure to select the region that matches your Checkmarx AST tenant.

### How It Works

The Checkmarx integration operates as follows:

1. **Project Discovery**: Pixee Enterprise Server discovers Checkmarx projects associated with your repositories
2. **Scan Retrieval**: The latest SAST scan results are fetched from the Checkmarx AST platform
3. **Vulnerability Analysis**: SAST vulnerabilities are converted to SARIF format and analyzed by Pixee's security analysis engine
4. **Fix Generation**: Pixee identifies applicable fixes for the discovered vulnerabilities
5. **Pull Request Creation**: Automatic fixes are applied and submitted as pull requests to the repository

The integration uses Checkmarx's REST API to retrieve project information, scan results, and vulnerability details.

## GitLab SAST Integration

GitLab SAST integration allows Pixee Enterprise Server to automatically consume SAST (Static Application Security Testing) scan results from your GitLab CI/CD pipelines and apply automated fixes to security vulnerabilities.

### Requirements

GitLab SAST integration requires:

- A GitLab personal access token with the following scopes:
  - `api`
  - `read_user`
  - `read_repository`
  - `read_api`
  - `write_repository`
  - `ai_features`
  - `read_registry`
  - `read_virtual_registry`
- A webhook secret for authenticating incoming pipeline notifications
- GitLab pipelines configured with SAST scanning (using GitLab's built-in SAST analyzer)

<Tip>
It is recommended to use a [GitLab service account](https://docs.gitlab.com/ee/user/profile/service_accounts.html){:target="_blank"} to generate the personal access token rather than a personal user account. Service accounts are not tied to individual users, which avoids disruption if a team member leaves or their account is modified. The service account should be granted access to the groups or projects that Pixee will manage.
</Tip>

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `SCM` section.

Select the `GitLab` checkbox to enable GitLab integration.

Enter the following information in the configuration fields:

- Base URI: The base URL of your GitLab instance (default: `https://gitlab.com`)
- Access Token: Your GitLab access token with the required scopes listed above
- Webhook Secret: Secret for webhook authentication

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CommandBlock>
platform:
  scm:
    gitlab:
      enabled: true
      baseUri: "https://gitlab.com" # or your GitLab instance URL
      token: "your-gitlab-access-token" # requires scopes: api, read_user, read_repository, read_api, write_repository, ai_features, read_registry, read_virtual_registry
      webhookSecret: "your-gitlab-webhook-secret"
      # Use existing secret instead of creating one
      existingSecret: ""
      secretKeys:
        # -- The secret key containing the token
        tokenKey: "token"
        # -- The secret key containing the webhookSecret
        webhookSecretKey: "webhookSecret"
</CommandBlock>

{{/if}}

### Webhook Configuration

To receive notifications when GitLab pipelines complete with SAST results, configure a webhook in your GitLab project or group settings.

#### Setting Up GitLab Webhooks

1. Navigate to your GitLab project page
2. Go to **Settings** > **Webhooks**
3. Add a new webhook with the following configuration:
   - **URL**: `https://<your-pixee-server.com>/api/v1/integrations/gitlab-default/webhooks`
   - **Secret Token**: Use the same webhook secret configured in Pixee Enterprise Server
   - **Trigger Events**: Select **Pipeline events**
   - **SSL verification**: Enable if using HTTPS (recommended)

The webhook secret configured in GitLab must match the webhook secret configured in your Pixee Enterprise Server.

### SAST Pipeline Configuration

Pixee Enterprise Server processes SAST results from two types of GitLab **Pipeline Events**:

1. **Branch Pipeline Events** - Triggered when pipelines run on the default branch (e.g., `main`, `develop`)
2. **Merge Request Pipeline Events** - Triggered when pipelines run specifically for merge requests

#### SAST Configuration with Advanced Security Features

GitLab provides built-in SAST analyzers including both Semgrep and GitLab Advanced SAST. To enable comprehensive SAST scanning that works with both branch and merge request pipelines, add the following to your `.gitlab-ci.yml`:

<CommandBlock>
workflow:
  # Run pipeline jobs on the pushes to the default branch and merge requests
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

include:
  - template: Jobs/SAST.gitlab-ci.yml

variables:
  GITLAB_ADVANCED_SAST_ENABLED: true

# Override specific SAST jobs to run on merge requests
semgrep-sast:
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

gitlab-advanced-sast:
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
</CommandBlock>

This configuration:

- **Runs pipelines** on merge request events and default branch pushes only
- **Enables GitLab Advanced SAST** in addition to the standard Semgrep-based SAST analyzer
- **Explicitly configures** both `semgrep-sast` and `gitlab-advanced-sast` jobs to run on merge requests
- **Ensures SAST coverage** for both Pixee's pull request hardening (from merge request pipelines) and repository-wide scanning (from default branch pipelines)

Pixee Enterprise Server will automatically detect and process SAST vulnerabilities from completed pipeline runs, applying fixes where possible and creating merge requests with the remediated code.

### How It Works

The GitLab SAST integration operates as follows:

1. **Pipeline Completion**: When a GitLab pipeline with SAST scanning completes, GitLab sends a webhook notification to Pixee Enterprise Server
2. **Vulnerability Retrieval**: Pixee fetches the SAST vulnerabilities from the GitLab API for the completed pipeline
3. **Analysis**: The vulnerabilities are converted to SARIF format and analyzed by Pixee's security analysis engine
4. **Fix Generation**: Pixee identifies applicable fixes for the discovered vulnerabilities
5. **Merge Request Creation**: Automatic fixes are applied and submitted as merge requests to the repository

Both regular branch pipelines and merge request pipelines are supported, with merge request pipelines triggering pull request hardening workflows for more targeted security improvements.

### Alternative: Using Pixee GitLab Component

As an alternative to the webhook-based configuration described above, you can use the Pixee GitLab component for a simplified integration that does not require webhook setup.

The Pixee GitLab component is available at [https://gitlab.com/pixee/pixee](https://gitlab.com/pixee/pixee){:target="\_blank"} and provides a pre-configured CI/CD component that handles delivering SAST findings to Pixee Enterprise Server directly from your pipeline, eliminating the need for webhook configuration.

For detailed configuration options, usage instructions, and requirements, refer to the component documentation at [https://gitlab.com/pixee/pixee](https://gitlab.com/pixee/pixee){:target="\_blank"}.

**Note**: When using the Pixee GitLab component, you do not need to configure GitLab webhooks as described in the "Webhook Configuration" section above.

## Datadog SAST Integration

Pixee Enterprise Server supports two methods for integrating with Datadog SAST:

- **Native integration** — connect Pixee directly to your Datadog account using API credentials configured in the admin console. Pixee fetches findings automatically.
- **SARIF upload** — run the open-source Datadog Static Analyzer CLI in your CI/CD pipeline and upload the SARIF output to Pixee via the API. This method does not require a Datadog account or API keys.

## Method 1: Native Integration

The native integration connects Pixee directly to your Datadog account. Configure it once in the admin console and Pixee will fetch SAST findings automatically without any CI/CD pipeline changes.

### Requirements

- A Datadog account with SAST enabled
- A Datadog API key and Application key

### Admin Console Configuration

1. In the admin console, navigate to the **Config** tab and find the **Security Tool Integrations** section.
2. Check the **Datadog** checkbox to enable the integration.
3. Enter your **API Key** in the field that appears.
4. Enter your **Application Key** in the field that appears.
5. Save the configuration and deploy the update.

This configures the `PIXEE_PLATFORM_INTEGRATIONS_DATADOG_API_KEY` and `PIXEE_PLATFORM_INTEGRATIONS_DATADOG_APPLICATION_KEY` environment variables on the platform deployment.

### Helm Configuration (Alternative)

If you prefer to configure the integration via Helm values instead of the admin console, set the following in your values file:

<CommandBlock>
platform:
  datadog:
    apiKey: "<your-api-key>"
    applicationKey: "<your-application-key>"
</CommandBlock>

Or reference an existing Kubernetes secret:

<CommandBlock>
platform:
  datadog:
    existingSecret: "my-datadog-secret"
    secretKeys:
      apiKeyKey: "apiKey"
      applicationKeyKey: "applicationKey"
</CommandBlock>

## Method 2: SARIF Upload via Datadog Static Analyzer CLI

This method uses the open-source [Datadog Static Analyzer CLI](https://github.com/DataDog/datadog-static-analyzer){:target="\_blank"} to scan your codebase and upload the resulting SARIF file to Pixee via the API. It does not require a Datadog account or API keys.

### Requirements

- The Datadog Static Analyzer CLI
- A Pixee authentication token. See [API Access](../api-access.md) for details.

### Installing the CLI

Compiled binaries of the Datadog Static Analyzer CLI can be found and downloaded from the [releases](https://github.com/DataDog/datadog-static-analyzer/releases){:target="\_blank"} page of its main Github repository. Find the release that matches the OS and architecture for the machine it will be running on.

### CLI Configuration

#### Filtering for security-only findings

The Datadog CLI applies many rule sets when scanning a codebase, many of which are not security related. To configure which rule sets are applied, the CLI reads a YAML file named `static-analysis.datadog.yml` in the current working directory that specifies which rules to use. You will need to create this YAML file if you wish to filter for security-only findings.

Here's an example of a YAML configuration file that only applies Java security rules:

<CommandBlock>
schema-version: v1
rulesets:
  - java-security
</CommandBlock>

Datadog's rulesets follow a consistent naming convention, so this pattern can be applied to projects that use other programming languages. For example, you can use `python-security` for a Python project. You can even specify multiple of these rulesets if the codebase that is being scanned contains multiple programming languages.

See Datadog's [SAST Rules](https://docs.datadoghq.com/security/code_security/static_analysis/static_analysis_rules/){:target="\_blank"} documentation for available rulesets.

### Manually running and uploading a scan to Pixee

Now that the CLI is installed and configured, it can be run in the desired codebase directory to generate a scan. To do this, invoke the following CLI command within the codebase's root directory:

<CommandBlock>
datadog-static-analyzer -i . -o ./report-sarif.json -f sarif
</CommandBlock>

The `-o` flag specifies the output scan file name and location. The `-f` flag specifies the output format. Pixee requires the SARIF output format. See the [README](https://github.com/DataDog/datadog-static-analyzer?tab=readme-ov-file#options){:target="\_blank"} for the CLI for a full list of options.

This output scan file can then be uploaded to Pixee via the API. To do this, you will need to retrieve the base URL and repository ID for the codebase you want to analyze. These can be extracted from the URL when opening the repository in Pixee Resolution Center. You can then send an HTTP POST request to the `/scans` endpoint for that repository using any HTTP client. Here's an example using cURL:

<CommandBlock>
curl -X POST "$BASE_URL/api/v1/repositories/$REPO_ID/scans" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $PIXEE_API_KEY" \
  -F 'file=@./report-sarif.json' \
  -F 'metadata={"tool":"datadog_sast","branch":"main"};type=application/json'
</CommandBlock>

This will cause Pixee to ingest all of the findings within this scan and automatically start to analyze them. You should be able to view progress on the analysis in the same repository page in Pixee Resolution Center.

### Automating scan and upload in your CI/CD pipeline

Any CI/CD platform can automate the Datadog SAST scan and upload workflow. The pipeline needs to perform three steps:

1. Install the Datadog Static Analyzer CLI
2. Run the scan, outputting SARIF
3. Upload the SARIF file to Pixee via the API

Make sure your `static-analysis.datadog.yml` configuration file is committed to the repository root so the CLI picks it up automatically during CI runs.

#### GitHub Actions example

Below is a complete GitHub Actions workflow that installs the Datadog Static Analyzer CLI, runs a scan, and uploads the results to Pixee. Add this file to your repository at `.github/workflows/datadog-sast-pixee.yml`.

The workflow uses three GitHub Actions secrets that you must configure in your repository settings:

- `PIXEE_API_KEY` — your Pixee API key
- `PIXEE_BASE_URL` — the base URL of your Pixee instance (e.g., `https://app.pixee.example.com`)
- `PIXEE_REPO_ID` — the repository ID from the Pixee Resolution Center URL

<CommandBlock>
name: Datadog SAST scan and upload to Pixee

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  datadog-sast:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v4

      - name: Install Datadog Static Analyzer CLI
        run: |
          ARCH=$(uname -m)
          case "$ARCH" in
            x86_64)  ARCH_NAME="x86_64" ;;
            aarch64) ARCH_NAME="aarch64" ;;
            arm64)   ARCH_NAME="aarch64" ;;
            *)       echo "Unsupported architecture: $ARCH"; exit 1 ;;
          esac

          LATEST=$(curl -s https://api.github.com/repos/DataDog/datadog-static-analyzer/releases/latest \
            | grep tag_name | cut -d '"' -f 4)

          curl -sL "https://github.com/DataDog/datadog-static-analyzer/releases/download/${LATEST}/datadog-static-analyzer-${ARCH_NAME}-unknown-linux-gnu.zip" \
            -o datadog-static-analyzer.zip
          unzip -o datadog-static-analyzer.zip -d /usr/local/bin
          chmod +x /usr/local/bin/datadog-static-analyzer
          rm datadog-static-analyzer.zip

      - name: Run Datadog SAST scan
        run: datadog-static-analyzer -i . -o results.sarif -f sarif

      - name: Upload SARIF to Pixee
        env:
          PIXEE_API_KEY: ${{ secrets.PIXEE_API_KEY }}
          PIXEE_BASE_URL: ${{ secrets.PIXEE_BASE_URL }}
          PIXEE_REPO_ID: ${{ secrets.PIXEE_REPO_ID }}
        run: |
          curl -X POST "$PIXEE_BASE_URL/api/v1/repositories/$PIXEE_REPO_ID/scans" \
            -H "Accept: application/json" \
            -H "Authorization: Bearer $PIXEE_API_KEY" \
            -F 'file=@./results.sarif' \
            -F 'metadata={"tool":"datadog_sast","branch":"${{ github.ref_name }}","workflow_execution_policy":"execute"};type=application/json'
</CommandBlock>

This workflow does not require Datadog API or App keys — it only uses the open-source static analyzer CLI and uploads results directly to Pixee.
