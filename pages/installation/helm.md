---
title: Helm Installation
visible_when:
  entitlements:
    - isHelmInstallEnabled
---

# Helm Installation

To install using Helm Deployment follow:

1. Authenticate against the Pixee Helm Registry:

   <CommandBlock>
   helm registry login registry.pixee.ai --username <your email address> --password <your license key>
   </CommandBlock>

2. **Preflight checks** - If there are any known issues that would prevent successful installation, the preflight checks will report them. To run the preflight checks:

   <CommandBlock>
   helm template oci://registry.pixee.ai/pixee/<release channel>/pixee-enterprise-server --values values.yaml | kubectl preflight -
   </CommandBlock>

   If there are no issues, or you are able to address all reported issues, continue with the installation using helm.

3. **Helm install** - Execute helm against the Kubernetes cluster to install, be sure to replace your release channel below (likely `stable` or `unstable`):
   <CommandBlock>
   helm upgrade --install pixee-enterprise-server oci://registry.pixee.ai/pixee/<release channel>/pixee-enterprise-server -f values.yaml -n pixee-enterprise-server --create-namespace
   </CommandBlock>

<Tip>
Be sure to replace `<release channel>` with your actual assigned channel, this is likely `stable` or `unstable`
</Tip>
