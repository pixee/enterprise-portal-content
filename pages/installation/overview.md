---
title: Installation Overview
---

# Installation Overview

Pixee Enterprise Server is a self-hosted solution that brings pixee.ai into a customer's infrastructure.

## Installation Methods

There are currently two methods available for installing Pixee Enterprise Server:

{{#if entitlements.isEmbeddedClusterDownloadEnabled}}

**Embedded Cluster**

This option provides the most streamlined installation, configuration, update, and support experience. This is the recommended method of installation for Pixee Enterprise Server as it provides a user-friendly interface for installation, configuration and enhanced troubleshooting capabilities.

{{/if}}

{{#if entitlements.isHelmInstallEnabled}}

**Helm Deployment**

This option allows users to deploy Pixee Enterprise Server into a managed kubernetes cluster.

{{/if}}
