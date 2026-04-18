---
title: Advanced Settings
---

# Advanced Settings

This section covers advanced configuration options for Pixee Enterprise Server.

## External Database

For production environments, it is recommended to configure an external database when deploying into production. See the [installation prerequisites](../installation/prerequisites.md#database) for requirements.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure external database in Embedded Cluster deployments follow:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Under `Database Type` select External.

Provide the following information:

- Database host
- Database port
- Database username
- Database password
- Database name

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure external database in Helm Deployment follow:

You can configure Pixee Enterprise Server to use an external database server. If you prefer to use the in-cluster, embedded database, you may skip this step.

To configure an external database, add the following to your `values.yaml`:

```yaml
platform:
  database:
    embedded: false
    host: "<your database hostname>"
    port: "<your database server port, defaults to 5432>"
    name: "<your database name, defaults to pixee_platform>"
    username: "<your database user with access to database>"
    password: "<your database user password>"
    # -- Use an existing secret for the password instead of passing directly in Values.  Secret must contain a `password` key.
    existingSecret: "<your postgres secret name>"
```

{{/if}}

## Embedded Database Credential Secrets

When using the embedded database (`platform.database.embedded: true`), Pixee Enterprise Server automatically creates Kubernetes secrets for CloudNative-PG managed database roles used by Superset and Authentik. If you prefer to manage these secrets yourself (e.g., via an external secrets operator), you can provide your own pre-existing secrets instead.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Embedded Cluster deployments manage these secrets automatically. No additional configuration is needed.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To use existing secrets for Superset and/or Authentik database credentials, first create the secret(s):

**Superset database credentials:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-superset-postgresql-credentials
type: kubernetes.io/basic-auth
stringData:
  username: "superset"
  password: "<your-password>"
```

**Authentik database credentials:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-authentik-postgresql-credentials
type: kubernetes.io/basic-auth
stringData:
  username: "authentik"
  password: "<your-password>"
```

Then reference them in your `values.yaml`:

```yaml
superset:
  database:
    existingSecret: "my-superset-postgresql-credentials"

authentik:
  database:
    existingSecret: "my-authentik-postgresql-credentials"
```

{{/if}}

## Object Store

### Embedded Object Store

You can configure Pixee Enterprise Server to use an embedded object store. This is the default configuration and is suitable for development and testing environments.

When embedded object store is enabled, you will also be prompted to configure the `Object Store Expiry` in `Days` for the object store. This setting determines how long objects will be retained in the embedded object store before they are automatically deleted.

### External Object Store

You can configure Pixee Enterprise Server to use an external object store. If you prefer to use the in-cluster, embedded object store, you may skip this step.

#### Requirements

The following are requirements of an external object store compatible with Pixee Enterprise Server:

- The object store and the Kubernetes cluster are able to communicate over the network
- The object store exposes a S3 compatible API
- A bucket has been created for use as the `pixee-analysis-input` bucket

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure external object store in Embedded Cluster deployments follow:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Under `Object Store Type` select External.

Provide the following information:

- Object store endpoint URL
- Object store username (Access Key ID)
- Object store password (Secret Access Key)
- Bucket name for pixee analysis inputs
- Bucket name for the pixee analysis service

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure external object store with static credentials in Helm Deployment follow:

#### Static Credentials

To configure an external object store, add the following to your `values.yaml`:

```yaml
global:
  pixee:
    objectStore:
      embedded: false
      endpoint: "<your object store endpoint url>"
      username: "<your object store username>" # Access Key ID for S3
      password: "<your object store password>" # Secret Access Key for S3
      credentialType: "static"
platform:
  inputBucket: "<your provisioned bucket name for pixee analysis inputs>"
analysis:
  objectStore:
    bucket: "<your provisioned bucket name for the pixee analysis service>"
```

#### Service Account Authentication

For enhanced security, you can use Kubernetes service account authentication instead of static credentials. This approach leverages cloud provider IAM roles and eliminates the need for long-lived access keys.

To configure service account authentication, add the following to your `values.yaml`:

```yaml
global:
  pixee:
    serviceAccount:
      create: false
      name: "your-external-service-account"
    objectStore:
      embedded: false
      endpoint: "<your object store endpoint url>"
      region: "<your object store region>"
      credentialType: "default"
      # username and password are not required with service account auth
platform:
  inputBucket: "<your provisioned bucket name for pixee analysis inputs>"
analysis:
  objectStore:
    bucket: "<your provisioned bucket name for the pixee analysis service>"
```

{{/if}}

## Git Clone Strategy

Pixee Enterprise Server supports two Git cloning strategies for VCS operations:

- **Partial Clone (Default)**: Downloads only the specific commit, tree, and blob objects needed for the requested revision. This provides optimal performance and minimal bandwidth usage but may not be supported by all Git servers.
- **Full Clone**: Downloads the complete repository including all history, branches, and objects. While this requires more time and bandwidth, it ensures maximum compatibility with all Git servers.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure the Git clone strategy in Embedded Cluster deployments:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Under `Git Clone Strategy`, select either:

- **Partial (recommended - faster, less bandwidth)**: For optimal performance
- **Full (maximum compatibility)**: For environments where partial clones are not supported

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure the Git clone strategy in Helm deployments, add the following to your `values.yaml`:

```yaml
platform:
  gitCloneStrategy: "partial" # or "full" for maximum compatibility
```

The default value is `partial` for optimal performance. Change to `full` if you encounter issues with Git servers that don't support partial clones.

{{/if}}

## Git Author Identity

When using a service account for Git operations (e.g., a GitLab service account), you may need to configure the author email and username that Pixee uses when creating commits. You can retrieve these values from your VCS provider's API:

```bash
# GitLab example — username
curl -s -H "Authorization: Bearer $GITLAB_TOKEN" "https://gitlab.com/api/v4/user" | jq '.username'

# GitLab example — email (separate endpoint)
curl -s -H "Authorization: Bearer $GITLAB_TOKEN" "https://gitlab.com/api/v4/user/emails" | jq '.[0].email'
```

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Under `Git Author Email` and `Git Author Username`, enter the service account's email and username.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

Add the following to your `values.yaml`:

```yaml
platform:
  gitAuthorEmail: "service-account@example.com"
  gitAuthorUsername: "pixee-bot"
```

{{/if}}

## Error Reporting

By default, Pixee Enterprise Server will send error and crash reports to Pixee via Sentry.io.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure error reporting in Embedded Cluster deployments follow:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section.

To disable automatic error and crash reporting, uncheck the error reporting option.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure error reporting in Helm Deployment follow:

To disable automatic error and crash reporting, add the following to your `values.yaml`:

```yaml
global:
  pixee:
    sentry:
      enabled: false
```

{{/if}}

## HTTP(S) Proxy Settings

You can configure Pixee Enterprise Server to route HTTP/HTTPS traffic through a proxy server.

### NO_PROXY Configuration Limitations

<Warning title="Important: NO_PROXY Wildcard Limitations">
The `NO_PROXY` environment variable in Pixee Enterprise Server does not support wildcard patterns. You must specify exact hostnames or IP addresses.

**Not Supported:**

```
NO_PROXY=*.internal.company.com
NO_PROXY=10.0.*.*
```

**Supported (Required Format):**

```
NO_PROXY=service1.internal.company.com,service2.internal.company.com,10.0.1.5,10.0.2.10
```

This limitation means that if you have multiple internal services that should bypass the proxy, each hostname must be explicitly listed in the `NO_PROXY` configuration.
</Warning>

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure HTTP(S) proxy in Embedded Cluster deployments follow:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section.

Select the `Configure HTTP(S) Proxy` checkbox if you need to route HTTP/HTTPS traffic through a proxy server. Once enabled, you can configure or modify the HTTP and HTTPS proxy server addresses as well as provide a comma separated list of domains to exclude from using your HTTP/HTTPS proxy.

**Note:** The NO_PROXY field requires exact hostnames or IP addresses. Wildcard patterns are not supported.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure HTTP(S) proxy in Helm Deployment follow:

Add the following to your `values.yaml`:

```yaml
global:
  pixee:
    httpProxy: "<address>:<port>" # HTTP proxy server host/address and port
    httpsProxy: "<address>:<port>" # HTTPS proxy server host/address and port
    noProxy: "<comma,separated,hosts>" # Comma separated list of exact hostnames/IPs to exclude from proxy (wildcards not supported)
```

**Example configuration:**

```yaml
global:
  pixee:
    httpProxy: "proxy.company.com:8080"
    httpsProxy: "proxy.company.com:8080"
    noProxy: "kubernetes.default,kubernetes.default.svc,10.0.0.1,database.internal.company.com,api.internal.company.com"
```

{{/if}}

## Private CA Certificates

If your environment uses self-signed certificates or a private Certificate Authority (CA), you can configure Pixee Enterprise Server to trust these certificates.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Embedded Cluster automatically detects and uses the host system's CA trust store at install time. Ensure your private CA certificates are installed on the host before running the Embedded Cluster installer — no additional Pixee configuration is needed.

**Updating CA certificates after installation:**

If you need to add new CA certificates after the initial installation:

1. Update the host's CA trust store (e.g. add the certificate and run `update-ca-trust`)
2. Wait up to one hour for the `kotsadm-private-cas` ConfigMap to refresh automatically, or force an immediate refresh:

   ```bash
   kubectl rollout restart deployment/embedded-cluster-operator -n embedded-cluster
   ```

3. Restart the Pixee platform deployment to pick up the new certificates:

   ```bash
   kubectl rollout restart deployment/pixee-platform -n <namespace>
   ```

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

Create a ConfigMap containing your PEM-encoded CA certificate(s):

```bash
kubectl create configmap my-ca-certs \
  --from-file=ca.pem=/path/to/your/ca-certificate.pem \
  -n <namespace>
```

Then reference it in your `values.yaml`:

```yaml
global:
  pixee:
    privateCACert: "my-ca-certs"
```

The ConfigMap may contain one or more PEM files, each with one or more certificates. All certificates will be imported into the trust stores used by the platform, analysis, and forge services.

**Updating CA certificates:**

To add or replace certificates, update the ConfigMap and restart the platform deployment:

```bash
kubectl rollout restart deployment/pixee-platform -n <namespace>
```

{{/if}}

<Warning title="Deprecated: Skip SSL Verification">
The `global.pixee.skipSSLVerification` Helm value (and the corresponding KOTS admin console checkbox) is deprecated and will be removed in a future release. This setting disables ALL certificate verification for outbound HTTPS connections, which is a security risk. Use private CA certificates instead.
</Warning>

## Host Aliases

You can configure a custom host-to-IP mapping (`/etc/hosts` entry) for the Platform pods. This is useful for environments with private DNS, split-horizon DNS, or services not resolvable via the cluster's DNS.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure host aliases in Embedded Cluster deployments:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Select the `Enable Custom Host Aliases` checkbox.

Once enabled, enter host aliases in `/etc/hosts` format in the text area — one entry per line, with an IP address followed by one or more space-separated hostnames. Lines starting with `#` are treated as comments and ignored.

**Example:**

```
10.0.0.1 service.internal api.internal
192.168.1.100 db.internal
```

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure host aliases in Helm deployments, add the following to your `values.yaml`:

```yaml
platform:
  hostAliases:
    - ip: "10.0.0.1"
      hostnames:
        - "service.internal"
        - "api.internal"
```

Each entry maps an IP address to one or more hostnames, which are added to the pod's `/etc/hosts` file.

{{/if}}

## Metrics Reporting

By default, Pixee Enterprise Server will send anonymized usage metrics to Pixee.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure metrics reporting in Embedded Cluster deployments follow:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section.

To disable metrics reporting, uncheck the metrics reporting option.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure metrics reporting in Helm Deployment follow:

To disable metrics reporting, add the following to your `values.yaml`:

```yaml
global:
  pixee:
    metrics:
      enabled: false
```

{{/if}}

## Object Store Signature Duration

By default, Pixee Enterprise Server generates pre-signed URLs for object store operations with a system-defined expiration duration. You can customize this duration by configuring the signature duration setting.

The duration should be specified as a string with time units. Common examples: "1h" (1 hour), "30m" (30 minutes), "2h" (2 hours), "45m" (45 minutes). If not configured, the system will use its default behavior.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure signature duration in Embedded Cluster deployments follow:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Locate the `Object Store Signature Duration` field.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure signature duration in Helm Deployment follow:

To configure a custom signature duration for pre-signed URLs, add the following to your `values.yaml`:

```yaml
platform:
  inputSignatureDuration: "<duration>"
```

{{/if}}

## Reverse Proxy Settings

If your Pixee Enterprise Server is accessible via a reverse proxy (i.e., ALB, App Gateway, NGINX, etc.), you may need to configure additional settings.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure reverse proxy settings in Embedded Cluster deployments follow:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section.

If you have an external system (i.e. App Gateway, Load Balancer, etc.) acting as a reverse proxy to your Pixee Enterprise Server, make sure you provide reverse proxy settings in the Advanced Settings section.

1. Check **Configure reverse proxy**
2. Select the **TLS Configuration** — choose where TLS is terminated:
   - **TLS at this cluster** — the cluster serves HTTPS and the reverse proxy forwards encrypted traffic
   - **TLS at reverse proxy** — the reverse proxy terminates TLS and forwards HTTP to the cluster
   - **No TLS (HTTP only)** — no encryption, not recommended for production
3. **Reverse Proxy Forwarded Headers** is enabled by default — this allows `X-Forwarded-Host`, `X-Forwarded-Port`, `X-Forwarded-Proto`, and `X-Forwarded-Ssl` headers to be forwarded to the application
4. Optionally enter the **Reverse Proxy Address Filter** to restrict which proxy IPs are trusted

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure reverse proxy settings in Helm Deployment follow:

Add the following applicable settings to your `values.yaml`:

```yaml
global:
  pixee:
    # Set externalProtocol when the reverse proxy terminates TLS and the cluster serves HTTP
    externalProtocol: "https" # protocol used by browsers (omit to use the same as protocol)
platform:
  proxy:
    enabled: true
    address: "<address of your proxy server, optional>"
    headers:
      forwarded: true|false # set to true to allow `Forwarded` header, defaults to false
      xForwarded: true|false # set to true to allow X-Forwarded-* headers, defaults to false
```

{{/if}}

### Non-Standard Port (Load Balancer)

If your load balancer exposes Pixee Enterprise Server on a non-standard port (e.g., port 5443 instead of 443), additional configuration is required so that OIDC authentication redirects include the correct port.

<Warning title="Required: Forwarded Headers">
Your load balancer **must** forward the port information to the application via one of these headers:

- `X-Forwarded-Host` with the port included (e.g., `X-Forwarded-Host: yourdomain.com:5443`), or
- `X-Forwarded-Port` as a separate header (e.g., `X-Forwarded-Port: 5443`)

This is how the application determines the correct port for constructing OIDC redirect URIs. Most load balancers (ALB, HAProxy, NGINX) set these automatically for non-standard ports.
</Warning>

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

1. Enable **Configure reverse proxy** (as described above)
2. Ensure **Reverse Proxy Forwarded Headers** is checked (enabled by default)
3. Enter the non-standard port number in the **Non-standard port** field (e.g., `5443`)

The non-standard port setting ensures that browser-facing OIDC redirect URLs (authorization endpoint, end-session endpoint) include the correct port. Combined with the `X-Forwarded-Host` header from your load balancer, the application will construct all redirect URIs with the correct host and port.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

Add the port to your `values.yaml` and enable proxy headers:

```yaml
global:
  pixee:
    port: "5443" # your non-standard port
platform:
  proxy:
    enabled: true
    headers:
      xForwarded: true
```

{{/if}}

<Note title="How it works">
The application uses two mechanisms to handle non-standard ports:

- **OIDC browser redirects** (authorization and end-session URLs): The configured port is included directly in the absolute URLs sent to the identity provider
- **OIDC callback redirect URI**: The application reads the `X-Forwarded-Host` or `X-Forwarded-Port` header from your load balancer to construct the `redirect_uri` parameter with the correct host and port

Both mechanisms are required for the full OIDC authentication flow to work correctly with a non-standard port.
</Note>

## Analysis Timeout Settings

Pixee Enterprise Server allows you to configure timeout values for different types of analyses. These settings control how long the platform waits for analysis results before timing out.

### Analysis Progress Timeout

The analysis progress timeout controls how long the platform waits for callbacks from the analysis service. If this duration passes without receiving an update, the analysis will timeout.

### SAST and SCA Analysis Timeouts

You can configure separate timeout values for SAST (Static Application Security Testing) and SCA (Software Composition Analysis) analyses. SCA analyses typically require longer timeouts than SAST due to dependency resolution complexity.

When not configured, the platform uses the general analysis timeout as the default for both analysis types.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure analysis timeouts in Embedded Cluster deployments:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section.

- **Analysis progress timeout**: Controls callback timeout from the analysis service (default: 15m)
- **SAST analysis timeout**: Timeout for SAST analyses (optional)
- **SCA analysis timeout**: Timeout for SCA analyses (optional, typically longer than SAST)

The duration should be specified as a string with time units. Common examples: "15m" (15 minutes), "30m" (30 minutes), "1h" (1 hour).

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure analysis timeouts in Helm deployments, add the following to your `values.yaml`:

```yaml
platform:
  analysisTimeout: "15m" # General analysis progress timeout
  sastAnalysisTimeout: "20m" # SAST-specific timeout (optional)
  scaAnalysisTimeout: "45m" # SCA-specific timeout (optional)
```

{{/if}}

## Agentic Triage Analyzer Strategy

Controls which AI reasoning approach is used for triage analysis across all rules.

Available strategies:

- **decision-tree** (default) — Autonomous agent that emulates a human-led triage process
- **react** — Multi-step reasoning with tool use
- **react-holistic** — Broader context analysis with tool use
- **claude-agent** — Claude-powered autonomous agent
- **anthropic-agent** — Anthropic SDK-based autonomous agent

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Select the desired strategy from the `Agentic Triage Analyzer Strategy` dropdown.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure the agentic triage analyzer strategy in Helm deployments, add the following to your `values.yaml`:

```yaml
analysis:
  agenticTriageAnalyzerStrategy: decision-tree
```

{{/if}}

## Project Context

When enabled, project-level contextual analysis is used to inform triage and fix analysis results. This allows the analysis engine to consider broader project context when evaluating findings, leading to more accurate triage decisions and higher quality fixes.

This setting is disabled by default.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Locate the `Enable Project Context` checkbox.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To enable project context in Helm deployments, add the following to your `values.yaml`:

```yaml
analysis:
  enableProjectContext: true
```

{{/if}}

## SCA Max Requests to Analyze

Controls the maximum number of requests to analyze during SCA (Software Composition Analysis). This limits the upper bound of dependency requests that SCA will process per analysis, helping to manage resource usage for repositories with large dependency trees.

The default value is `5`.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Locate the `SCA Max Requests to Analyze` field and enter the desired value.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure the SCA max requests to analyze in Helm deployments, add the following to your `values.yaml`:

```yaml
analysis:
  scaMaxRequestsToAnalyze: 5
```

{{/if}}

## SCA Exploitability Fix Shortcircuit

When enabled, fix generation is skipped for findings that SCA (Software Composition Analysis) determines are not exploitable. This reduces unnecessary compute usage by not generating fixes for vulnerabilities that cannot be reached in practice.

This setting is disabled by default.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Locate the `Skip fix for non-exploitable SCA findings` checkbox.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To enable this optimization in Helm deployments, add the following to your `values.yaml`:

```yaml
analysis:
  useScaExploitabilityToShortcircuitFix: true
```

{{/if}}

## Vendored File Triage

When enabled, a specialized triage strategy is used for vendored files. This improves the accuracy of analysis results for files that are vendored (copied from external sources) rather than authored in-house.

This setting is enabled by default.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Locate the `Enable vendored file triage` checkbox.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

This is enabled by default. To disable it in Helm deployments, add the following to your `values.yaml`:

```yaml
analysis:
  enableVendoredFileTriage: false
```

{{/if}}

## Analysis Input Caching

The analysis service supports URL-based input caching to improve performance for repeated analyses. When enabled, downloaded analysis inputs are cached locally to avoid redundant downloads.

Caching is enabled by default with a 24-hour TTL and 10GB maximum cache size.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section.

- **Enable analysis input caching**: Toggle to enable or disable caching (enabled by default)
- **Analysis cache TTL (seconds)**: Set the time-to-live for cached inputs (default: 86400 seconds / 24 hours)

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure analysis input caching in Helm deployments, add the following to your `values.yaml`:

```yaml
analysis:
  cache:
    enabled: true
    defaultTtlSeconds: 86400 # 24 hours
    maxSizeBytes: 10737418240 # 10GB
    honorCacheControl: true
```

{{/if}}

## Analysis Backpressure

The analysis backpressure feature enables the analysis service to proactively cancel analyses that cannot successfully complete within platform timeout limits. This helps prevent wasted compute resources on analyses that would eventually timeout.

This setting is enabled by default.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure analysis backpressure in Embedded Cluster deployments:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Locate the `Enable analysis backpressure` checkbox.

- **Enabled (default)**: The analysis service will proactively cancel analyses that cannot complete in time
- **Disabled**: Analyses will run until they complete or timeout naturally

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure analysis backpressure in Helm deployments, add the following to your `values.yaml`:

```yaml
analysis:
  backpressureEnabled: true # or false to disable
```

{{/if}}

## Transitive Dependency Analysis

Transitive dependency analysis enables deeper vulnerability detection by analyzing indirect (transitive) dependencies during SCA (Software Composition Analysis). When enabled, the analysis service will trace dependency chains beyond direct dependencies to identify vulnerabilities in the full dependency tree.

This setting is disabled by default.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

To configure transitive dependency analysis in Embedded Cluster deployments:

Navigate to the admin console, select the `Config` tab, then go to the `Advanced Settings` section. Locate the `Enable Transitive Dependency Analysis` checkbox.

- **Enabled**: The analysis service will analyze transitive dependencies during SCA
- **Disabled (default)**: Only direct dependencies are analyzed

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

To configure transitive dependency analysis in Helm deployments, add the following to your `values.yaml`:

```yaml
analysis:
  enableTransitiveDependencyAnalysis: true # or false to disable (default)
```

{{/if}}
