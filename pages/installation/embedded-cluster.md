---
title: Embedded Cluster Installation
visible_when:
  entitlements:
    - isEmbeddedClusterDownloadEnabled
---

# Embedded Cluster Installation

To install using Embedded Cluster follow:

1. From your virtual machine, download the Pixee installer:

   <CommandBlock>
   curl -f "https://distribution.pixee.ai/embedded/pixee/<release channel>" -H "Authorization: <your license ID>" -o pixee.tgz
   </CommandBlock>

2. Extract the Pixee installer:

   <CommandBlock>
   tar -xvzf pixee.tgz
   </CommandBlock>

3. Run the Pixee installer:

   <CommandBlock>
   sudo ./pixee install --license license.yaml
   </CommandBlock>

    <Note>
    The directory used for data storage can be changed by passing the --data-dir
    </Note>

4. You will be prompted to set an admin password, this password will grant access to the admin console later

5. When the installer completes, visit the admin console url in your browser: `https://<domain name or vm ip>:30000`
   - You may receive a self-signed certificate warning from your browser, this is expected
   - If you have a domain name and TLS certificate available you can configure the admin console to use them by following the prompts, or you can

6. The admin console will then load the configuration page. You will be directed through a workflow that will step you through configuring Pixee Enterprise Server.

<Tip>
Be sure to replace `<release channel>` with your actual assigned channel, this is likely `stable` or `unstable`
</Tip>
