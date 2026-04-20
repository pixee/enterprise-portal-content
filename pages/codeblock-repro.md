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

## Reference — CommandBlock preserves indentation

Identical content inside `<CommandBlock>` keeps its indentation (the rendered React Server Component payload shows children serialized to a base64 `encoded` prop). `<CommandBlock>` does not highlight, and its `language=` prop is also not visible in rendered output, but indentation works.

<CommandBlock>
global:
  pixee:
    localMetrics:
      enabled: true
</CommandBlock>

## Asks

1. What is the canonical source pattern for passing multi-line content to `<CodeBlock>` so that indentation is preserved?
2. Is `language=` highlighting wired up on this portal version, and if so, what values are accepted?
