---
title: Pixee Enterprise Server Frequently Asked Questions
---

# Pixee Enterprise Server Frequently Asked Questions

Below are list of frequently asked questions related to Pixee Enterprise Server installation, configuration and operation. If your question is not answered below, please [let us know](mailto:support@pixee.ai)!

### How do I view the pods and logs?

For detailed instructions on viewing pods and logs, see the [Observability - Logs & Debugging](observability/logs.md) section.

### How do I verify that the GitHub App is successfully sending events to pixee-enterprise-server?

To confirm that events are being properly transmitted from the GitHub App to `pixee-enterprise-server`, you can review the GitHub App's event log. This log can be accessed through your GitHub App's settings under the "Advanced" section.

GitHub provides detailed documentation on viewing webhook deliveries [here](https://docs.github.com/en/webhooks/testing-and-troubleshooting-webhooks/viewing-webhook-deliveries).

The event log displays a comprehensive list of all events sent from the GitHub App, along with their respective response codes.

### How do I retrieve a file out of SeaweedFS?

Pixee uses SeaweedFS to store CodeTF files and we may ask for those files to help debug issues related to Pixee.

CodeTF files are an interchange format used by the Pixee platform. They represent the results of fixes and therefore can be useful for debugging fix-related issues.

SeaweedFS ports are not exposed by default. To access the SeaweedFS web UI, first create an SSH tunnel from your local machine:

<CommandBlock>
ssh -L 8888:localhost:8888 pixee@<ip>
</CommandBlock>

Then in the SSH session, use the `kubectl port-forward` command to forward the port:

<CommandBlock>
kubectl port-forward -n kotsadm svc/pixee-enterprise-server-seaweedfs-filer 8888:8888
</CommandBlock>

This will forward the SeaweedFS web UI to port 8888. Remember to kill these commands once you are done. You can then access the SeaweedFS web UI by visiting `http://localhost:8888` in your browser to browse and download files.

### How do I update the Pixee Enterprise Server embedded cluster admin console domain name and TLS certificate?

If you have registered a domain name for your Pixee Enterprise Server and have a valid TLS certificate and want to
replace the self-signed certificate used by the admin console during the initial installation, follow these steps:

1. (optional) Retrieve the TLS certificate and private key from the cluster (skip this step if you already have the cert and key available)
   <CommandBlock>
   # From the Pixee Enterprise Server virtual machine
   sudo ./pixee shell
   kubectl get secret pixee-platform-tls-requested -o jsonpath='{.data.tls\.crt}' -n kotsadm | base64 --decode > cert.pem
   kubectl get secret pixee-platform-tls-requested -o jsonpath='{.data.tls\.key}' -n kotsadm | base64 --decode > key.pem
   </CommandBlock>
   Download the retrieved TLS certificate and private key locally
   <CommandBlock>
   # From your local machine
   scp pixee@<vm ip address>:/home/pixee/cert.pem ./
   scp pixee@<vm ip address>:/home/pixee/key.pem ./
   </CommandBlock>
2. Prepare the existing admin console tls secret to be updated

   <CommandBlock>
   # From the Pixee Enterprise Server virtual machine
   sudo ./pixee shell
   kubectl -n default annotate secret kotsadm-tls acceptAnonymousUploads=1 --overwrite -n kotsadm
   PROXY_SERVER=$(kubectl get pods -A | grep kurl-proxy | awk '{print $2}')
   kubectl delete pods $PROXY_SERVER -n kotsadm
   </CommandBlock>

   > ℹ️ The `acceptAnonymousUploads` annotation will be removed after completing the update in the next step

3. In a browser visit `https://<vm ip addresss>:30000/tls` and follow the prompts to set the admin console domain name to match the existing Pixee Enterprise Server and upload the TLS certificate and private key
4. When completed you should be able to browse to the admin console at `https://<domain name>:30000`

### How do I troubleshoot Pixee analysis failures or limited results?

The first step in troubleshooting is to generate a support bundle.

To generate a support bundle for an Embedded Cluster installation:

1. Click `Troublshoot` in the navigation bar of the Pixee Enterprise Server Admin Console
2. Click `Generate a Support Bundle`
3. Click `Analyze`
4. Wait for the support bundle to generate, this may take a few minutes.

When the support bundle is ready, you will be directed to a report. Any detected issues will be highlighted in the report with suggested next steps. If none of the suggested steps resolve your issue, contact Pixee for further support.

To generate a support bundle for a Helm CLI installation:

1. run `kubectl support-bundle --load-cluster-specs`

A Helm CLI installation support bundle will generate an archive you can share with Pixee for further support.

### How do I troubleshoot Authentik blueprint errors?

Pixee Enterprise Server uses [Authentik blueprints](https://docs.goauthentik.io/developer-docs/blueprints/) to declaratively configure the OIDC provider, application, and brand settings. Blueprints are automatically discovered and applied by the Authentik worker pod on a periodic cycle (~60 seconds). If a blueprint fails to apply, it enters an `error` state and will not be retried until the issue is resolved.

#### Checking Blueprint Status

Find the Authentik worker pod and check blueprint status:

<CommandBlock>
# Find the worker pod
kubectl get pods -n <namespace> -l app.kubernetes.io/name=authentik,app.kubernetes.io/component=worker

# Check all blueprint statuses
kubectl exec -n <namespace> <worker-pod> -- ak shell -c "
from authentik.blueprints.models import BlueprintInstance
for bp in BlueprintInstance.objects.all():
    print(f'{bp.name} | status={bp.status} | enabled={bp.enabled} | path={bp.path}')
"
</CommandBlock>

Blueprint statuses:

- **`successful`** — Applied without errors
- **`error`** — Failed to apply; will not be retried automatically
- **`unknown`** — Not yet processed or was manually reset

#### Finding the Error

When a blueprint is in `error` state, the status alone does not show the error message. To see the actual error, manually trigger a blueprint apply from the worker pod:

<CommandBlock>
kubectl exec -n <namespace> <worker-pod> -- ak apply_blueprint mounted/cm-pixee-authentik-blueprint/pixee-oidc.yaml
</CommandBlock>

This will output the full error details, such as serializer validation errors or missing references.

#### Fixing and Reapplying

1. **Fix the root cause** (e.g., correct an invalid field value in the blueprint configmap)
2. **Deploy the fix** so the updated configmap is mounted in the worker pod
3. **Reset the blueprint status** if it's stuck in `error`:

   <CommandBlock>
   kubectl exec -n <namespace> <worker-pod> -- ak shell -c "
   from authentik.blueprints.models import BlueprintInstance
   bp = BlueprintInstance.objects.get(name='Pixee OIDC Provider')
   bp.status = 'unknown'
   bp.save()
   "
   </CommandBlock>

4. Wait for the next discovery cycle (~60 seconds), or manually trigger the apply:

   <CommandBlock>
   kubectl exec -n <namespace> <worker-pod> -- ak apply_blueprint mounted/cm-pixee-authentik-blueprint/pixee-oidc.yaml
   </CommandBlock>

For more details on blueprint troubleshooting, see the [Authentik blueprint documentation](https://docs.goauthentik.io/developer-docs/blueprints/).

### What internet access is necessary for Pixee Enterprise Server?

Pixee Enterprise Server will need access to certain resources on the internet during installation/upgrade and normal operation. All _optional_ resources can be disabled via config/values. If you need complete air gap support, please contact us. The resources are detailed below:

#### During installation/update:

| Domain                                        | IP Addresses (if available)                                                                                                 | Notes                                 |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| images.pixee.ai<br/>proxy.replicated.com      | see https://github.com/replicatedhq/ips/blob/main/ip_addresses.json#L52-L57                                                 | Pixee container image registry        |
| registry.pixee.ai<br/>registry.replicated.com | see https://github.com/replicatedhq/ips/blob/main/ip_addresses.json#L20-L25                                                 | Pixee Helm registry                   |
| github.com                                    | see api section at https://api.github.com/meta                                                                              | GitHub integration (_optional_)       |
| api.openai.com                                | N/A                                                                                                                         | OpenAI integration (_optional_)       |
| \*.api.letsencrypt.org                        | see [Letsencrypt FAQ](https://letsencrypt.org/docs/faq/#what-ip-addresses-does-let-s-encrypt-use-to-validate-my-web-server) | TLS certificate requests (_optional_) |

#### During normal operation:

| Domain                                   | IP Addresses (if available)                                                             | Notes                           |
| ---------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------- |
| distribution.pixee.ai<br/>replicated.app | 162.159.133.41<br/>162.159.134.41<br/>2606:4700:7::a29f:8529<br/>2606:4700:7::a29f:8629 | Metrics reporting               |
| sentry.io<br/>\*.us.sentry.io            | 35.186.247.156/32<br/>34.120.195.249/32                                                 | Crash reporting (_optional_)    |
| github.com                               | see api section at https://api.github.com/meta                                          | GitHub integration (_optional_) |
| api.openai.com                           | N/A                                                                                     | OpenAI integration (_optional_) |
