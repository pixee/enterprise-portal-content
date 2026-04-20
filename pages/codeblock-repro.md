---
title: CodeBlock Repro
---

# CodeBlock rendering repro

On this portal, `<CodeBlock>` does not preserve indentation in multi-line content and no `language=` syntax highlighting is applied. The docs at https://docs.replicated.com/vendor/enterprise-portal-configure describe `<CodeBlock>` as "Display multi-line code blocks with syntax highlighting" and list a `language` prop, but do not show an example of `<CodeBlock>` with multi-line indented content, so it's unclear what the expected source pattern is.

## Attempt — raw children

**Source:**

<CommandBlock>
<CodeBlock language="yaml">
global:
  pixee:
    localMetrics:
      enabled: true
</CodeBlock>
</CommandBlock>

**Rendered:**

<CodeBlock language="yaml">
global:
  pixee:
    localMetrics:
      enabled: true
</CodeBlock>

**Observed:**

- YAML indentation (2-space nesting) is lost; output is flat.
- No syntax-highlighting tokens in the rendered `<pre><code>`.
- The `language="yaml"` prop has no visible effect.

## Attempt 2 — pattern observed in another content repo

The only public fork of `replicatedhq/enterprise-portal-content` that uses `<CodeBlock>` is <a href="https://github.com/PlotNotes/enterprise-portal-content/blob/main/pages/installation/helm-chart-reference.md" target="_blank" rel="noopener noreferrer">PlotNotes/enterprise-portal-content</a>. It uses the same raw-children pattern plus an undocumented `title=` prop (not listed in the Replicated docs prop table).

**Source (verbatim from PlotNotes repo):**

<CommandBlock>
<CodeBlock language="yaml" title="custom-values.yaml">
global:
  domain: adopt.example.com

postgresql:
  enabled: false
  externalHost: db.example.com
  auth:
    existingSecret: purrfect-db-credentials

ingress:
  hostname: adopt.example.com
  tls:
    enabled: true
    secretName: adopt-tls

app:
  replicas: 3
</CodeBlock>
</CommandBlock>

**Rendered:**

<CodeBlock language="yaml" title="custom-values.yaml">
global:
  domain: adopt.example.com

postgresql:
  enabled: false
  externalHost: db.example.com
  auth:
    existingSecret: purrfect-db-credentials

ingress:
  hostname: adopt.example.com
  tls:
    enabled: true
    secretName: adopt-tls

app:
  replicas: 3
</CodeBlock>

## Reference — CommandBlock preserves indentation

Identical content inside `<CommandBlock>` keeps its indentation (the rendered React Server Component payload shows children serialized to a base64 `encoded` prop). `<CommandBlock>` does not highlight, and its `language=` prop is also not visible in rendered output, but indentation works.

<CommandBlock>
global:
  pixee:
    localMetrics:
      enabled: true
</CommandBlock>

## Asks

1. What is the canonical source pattern for passing multi-line content to `<CodeBlock>` so that indentation is preserved? The docs' Example section at https://docs.replicated.com/vendor/enterprise-portal-configure never demonstrates `<CodeBlock>` (only `<CommandBlock>`), and the one public content-repo usage we found (PlotNotes, linked above) uses the same raw-children pattern shown in Attempt 1.
2. Is `language=` syntax highlighting wired up on this portal version, and if so, what values are accepted?
3. Is `title=` a supported prop on `<CodeBlock>`? It's used in the PlotNotes repo but isn't listed in the docs' prop table.
