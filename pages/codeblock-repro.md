---
title: CodeBlock Repro
---

# CodeBlock rendering repro

This page demonstrates that `<CodeBlock>` on the Enterprise Portal does not produce the behavior documented at https://docs.replicated.com/vendor/enterprise-portal-configure — neither with raw JSX children nor with the template-literal pattern shown in the official docs example.

## Attempt 1 — raw JSX children

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

**Expected:** indented YAML with syntax highlighting.
**Actual:** indentation is collapsed; no highlighting.

## Attempt 2 — JSX template literal (docs pattern)

The docs CommandBlock example uses `{` `` ` `` `...` `` ` `` `}` to preserve whitespace. Applying the same pattern to CodeBlock:

**Source:**

<CommandBlock>
<CodeBlock language="yaml">
  {`global:
  pixee:
    localMetrics:
      enabled: true`}
</CodeBlock>
</CommandBlock>

**Rendered:**

<CodeBlock language="yaml">
  {`global:
  pixee:
    localMetrics:
      enabled: true`}
</CodeBlock>

**Expected:** the `{` `` ` `` `...` `` ` `` `}` JSX expression is evaluated; newlines and indentation are preserved; syntax highlighting is applied.
**Actual:** the `{` and `}` render as literal characters, and newlines collapse to single spaces. The portal's MDX compiler appears not to evaluate JSX expressions in component children.

## Reference — CommandBlock works

Identical content inside `<CommandBlock>` preserves indentation (via a base64 `encoded` prop observed in the rendered React Server Component payload), though `language=` has no visible effect:

<CommandBlock language="yaml">
global:
  pixee:
    localMetrics:
      enabled: true
</CommandBlock>
