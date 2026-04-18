---
title: Development Platform Integrations
---

# Development Platform Integrations

This section covers integrating Pixee Enterprise Server with various development platforms and source code management systems.

## Azure DevOps Integration

Azure DevOps integration allows Pixee Enterprise Server to work with your Azure DevOps repositories and requires a personal access token with specific permissions.

### Requirements

Azure DevOps integration requires:

- Your Azure DevOps organization name
- A personal access token with a custom scope that includes full _Code_ access (not "Full access" which grants broader permissions than necessary)

<Note>
The webhook user and password are optional properties for Azure DevOps webhook authentication. If configured, these credentials will be used to authenticate incoming webhook requests from Azure DevOps.
</Note>

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Development Platforms` section.

Select the `Azure DevOps` checkbox to enable Azure DevOps integration.

Enter the following information in the configuration fields:

- Organization: Your Azure DevOps organization name
- Token: Your personal access token with full _Code_ access
- Webhook credentials (optional): Username and password for webhook authentication if desired

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CodeBlock language="yaml">
platform:
  scm:
    azure:
      organization: "<your azure devops organization name>"
      token: "<your personal access token>"
      # Optional: For webhook authentication
      # webhook:
      #   user: "<your webhook username>"
      #   password: "<your webhook password>"
      # Use existing secret instead of creating one
      existingSecret: ""
      secretKeys:
        # -- The secret key containing the token
        tokenKey: "token"
        # -- The secret key containing the webhook password
        webhookPasswordKey: "webhookPassword"
</CodeBlock>

{{/if}}

## BitBucket Cloud Integration

BitBucket Cloud integration allows Pixee Enterprise Server to work with your BitBucket repositories and requires account credentials with specific permissions.

For security, it is recommended to create and use an API token for BitBucket Cloud integration rather than using personal credentials. See the [BitBucket API Token documentation](https://support.atlassian.com/bitbucket-cloud/docs/create-a-repository-access-token/){:target="\_blank"} for information on creating an API token.

<Note>
BitBucket API tokens require your account's **email address** for API authentication, while Git operations use your **username**. Make sure to configure both values.
</Note>

### Requirements

BitBucket Cloud integration requires:

- A BitBucket Cloud username (used for Git operations)
- Your BitBucket account email address (used for API authentication)
- An API token with the following scopes:
  - `read:user:bitbucket`
  - `read:workspace:bitbucket`
  - `read:repository:bitbucket`
  - `read:pullrequest:bitbucket`
  - `write:repository:bitbucket`
  - `write:pullrequest:bitbucket`

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Development Platforms` section.

Select the `BitBucket` checkbox to enable BitBucket Cloud integration.

Enter the following information in the configuration fields:

- Username: Your BitBucket Cloud username (used for Git operations)
- Email Address: Your BitBucket account email address (used for API authentication)
- API Token: Your BitBucket API token

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CodeBlock language="yaml">
platform:
  scm:
    bitbucket:
      username: "<your bitbucket cloud username>"
      emailAddress: "<your bitbucket account email address>"
      apiToken: "<your bitbucket api token>"
      # Use existing secret instead of creating one
      existingSecret: ""
      secretKeys:
        # -- The secret key containing the API token
        apiTokenKey: "apiToken"
</CodeBlock>

{{/if}}

## GitHub Integration

GitHub integration allows Pixee Enterprise Server to work with GitHub.com or self-hosted GitHub Enterprise Servers and requires a custom GitHub app to be created.

Pixee Enterprise Server is able to integrate with GitHub.com and self-hosted GitHub Enterprise Servers. If you are self-hosting a GitHub enterprise server or otherwise have configured GitHub enterprise server on a domain other than github.com, see the [Configuration](#configuration) section below for instructions on setting your custom GitHub domain.

GitHub integration comes in the form of a custom GitHub app, which will be needed to configure GitHub integration in Pixee Enterprise Server. A custom GitHub app configures webhook events, event destination, and permissions for enhanced GitHub integration. In creating this application, we have followed the best [practices provided by GitHub](https://docs.github.com/en/apps/creating-github-apps/about-creating-github-apps/best-practices-for-creating-a-github-app){:target="\_blank"}.

<Note>
Network communication between your GitHub (.com or Enterprise Server) and Pixee Enterprise Server must exist. This can vary based on the deployment configuration of GitHub Enterprise Server and Pixee Enterprise Server.
</Note>

### GitHub App Setup

Unless otherwise instructed, leave the existing default values provided by GitHub.

1. Go to [https://github.com/settings/apps](https://github.com/settings/apps){:target="\_blank"}, replace `github.com` with your own private GitHub host as needed.
1. Click `New GitHub App` button.
1. Set the `GitHub App name` to something unique (i.e. "AcmePixeebotApp"), save this value for later.
1. Set `Homepage URL` to anything (i.e. "https://pixee.ai"), this can be updated later.
1. Set the `Callback URL` to the URL of your host/cluster in the following format [http://acme.getpixee.com/api/auth/login](http://acme.getpixee.com/api/auth/login).
1. Check `Request user authorization (OAuth) during installation`.
1. Check `Active` under `Webhook`.
1. Set `Webhook URL`, to the URL of your host/cluster in the following format [http://acme.getpixee.com/github-event](http://acme.getpixee.com/github-event).
1. Set `Webhook Secret` to a secret value, a randomly generated string will work (save this for later).
1. Set these `Repository permissions`:

   | Repository permissions | Access         |
   | ---------------------- | -------------- |
   | Checks                 | Read and write |
   | Code scanning alerts   | Read and write |
   | Commit statuses        | Read and write |
   | Contents               | Read and write |
   | Dependabot alerts      | Read and write |
   | Issues                 | Read and write |
   | Metadata               | Read-only      |
   | Pull Requests          | Read and write |
   | Workflows              | Read and write |

1. Set these `Organization permissions`:

   | Organization permission | Access    |
   | ----------------------- | --------- |
   | Members                 | Read-only |

1. Set these `Account permissions`:

   | Account permissions | Access    |
   | ------------------- | --------- |
   | Email addresses     | Read-only |

1. Check to `Subscribe to events` for the following:
   - Code scanning alert
   - Check Run
   - Create
   - Dependabot alert
   - Issue Comment
   - Issues
   - Pull request
   - Pull request review
   - Pull request review comment
   - Pull request review thread
   - Push
   - Repository

1. For `Where can this GitHub App be installed?` select `Only on this account`, this can be updated later.
1. Click `Create GitHub App` button.
1. Once the GitHub App is created, you should see the GitHub App configuration page.
1. Copy `App ID` and save for later.
1. Scroll down and click `Generate a private key`, download the private key file and save for later.

### Configuration

Select your installation method for instructions.

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster Deployment**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Development Platforms` section.

Select the `GitHub` checkbox to enable GitHub integration.

If you are self-hosting a GitHub enterprise server or otherwise have configured GitHub enterprise server on a domain other than github.com, be sure to select `custom domain` for the `GitHub domain` setting in the Pixee Enterprise Server admin console and enter your custom GitHub domain.

After creating up your GitHub App, insert the following data into the appropriate fields on the Pixee Enterprise Server admin console configuration screen:

- app name
- app id
- app private key (downloaded from browser)

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CodeBlock language="yaml">
platform:
  github:
    appName: "<your custom GitHub app name>"
    appId: "<your custom GitHub app id>"
    appWebhookSecret: "<your custom GitHub app webhook secret>"
    appPrivateKey: |
      -----BEGIN RSA PRIVATE KEY-----
      ...
      -----END RSA PRIVATE KEY-----
    # -- Use an existing secret instead of creating one
    existingSecret: ""
    secretKeys:
      # -- The secret key containing the appWebhookSecret
      appWebhookSecretKey: appWebhookSecret
      # -- The secret key containing the appPrivateKey
      appPrivateKeySecretKey: appPrivateKey
    # For GitHub Enterprise hosted at domains other than github.com, uncomment set your GitHub Enterprise url:
    # url: "https://github.your-company.com"
</CodeBlock>

<Tip>
Be sure to check the indentation is correct for each line of the GitHub app private key
</Tip>

{{/if}}

### Verification

If you enabled GitHub integration and created a custom GitHub app, you can verify your GitHub App connectivity by checking your GitHub App's event log.
This log can be accessed through your GitHub App's settings under the "Advanced" section. See [GitHub.com](https://docs.github.com/en/webhooks/testing-and-troubleshooting-webhooks/viewing-webhook-deliveries#viewing-deliveries-for-github-app-webhooks){:target="\_blank"} for more information.

## GitLab Integration

Pixee Enterprise Server is able to integrate with `https://gitlab.com` as well as self-hosted GitLab servers. If you have a self-hosted GitLab server, see the [Configuration](#configuration) section below for instructions on setting your custom GitLab base URI.

### Requirements

GitLab integration requires:

- A GitLab personal access token with the following scopes:
  - `api`
  - `read_user`
  - `read_repository`
  - `read_api`
  - `write_repository`
  - `ai_features`
  - `read_registry`
  - `read_virtual_registry`
- (Optional) Self-hosted GitLab server base URI if not using GitLab.com
- (Optional) Webhook secret for GitLab webhook integration

<Tip>
It is recommended to use a [GitLab service account](https://docs.gitlab.com/ee/user/profile/service_accounts.html){:target="_blank"} to generate the personal access token rather than a personal user account. Service accounts are not tied to individual users, which avoids disruption if a team member leaves or their account is modified. The service account should be granted access to the groups or projects that Pixee will manage.
</Tip>

### Configuration

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

For embedded cluster deployments, navigate to the admin console, `Config` tab and then to the `Development Platforms` section.

Select the `GitLab` checkbox to enable GitLab integration.

Enter the following information in the configuration fields:

- Token: Your GitLab personal access token with the required scopes listed above
- Base URI (optional): Your self-hosted GitLab server URL
- Webhook secret (optional): Secret for webhook authentication

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

For Helm deployments, add the following to your `values.yaml`:

<CodeBlock language="yaml">
platform:
  scm:
    gitlab:
      # For self-hosted GitLab, add:
      # baseUri: "https://gitlab.your-company.com"
      token: "your-personal-access-token" # requires scopes: api, read_user, read_repository, read_api, write_repository, ai_features, read_registry, read_virtual_registry
      # If you are using GitLab webhooks, provide the webhook secret:
      # webhookSecret: "your-gitlab-webhook-secret"
      # Use existing secret instead of creating one
      existingSecret: ""
      secretKeys:
        # -- The secret key containing the token
        tokenKey: "token"
        # -- The secret key containing the webhookSecret
        webhookSecretKey: "webhookSecret"
</CodeBlock>

{{/if}}

### Webhook Configuration

If you want to use webhooks to notify Pixee of build events, you'll need to configure webhooks in your GitLab project.

The webhook URI should be: `https://<example-pixee-server.com>/api/v1/integrations/gitlab-default/webhooks`

For detailed instructions on configuring GitLab webhooks, see the [GitLab Webhook Documentation](https://docs.gitlab.com/user/project/integrations/webhooks/){:target="\_blank"}.

The webhook secret configured in Pixee Enterprise Server should match the secret token configured in your GitLab webhook settings.
