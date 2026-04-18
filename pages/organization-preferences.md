---
title: Organization Preferences
---

# Organization Preferences

Organization preferences let you define natural-language guidelines that Pixee applies across all repositories in your organization. Instead of configuring each repository individually, you can set baseline rules once and have them take effect everywhere.

## Accessing Preferences

Select your organization from the top bar in the Pixee Platform UI, then navigate to **Preferences**. The preferences editor supports Markdown formatting and saves your changes immediately.

## Writing Preferences

Write your preferences in Markdown. Each preference should clearly describe a rule or guideline for how Pixee should handle findings and generate fixes across your repositories.

**Examples of effective preferences:**

- "Ignore findings in test directories"
- "Apply stricter checks to repositories with public visibility"
- "Prefer `slf4j` over `java.util.logging` when suggesting logging changes"
- "Do not suggest changes to files in `src/generated/`"

<Tip>
Start with a few high-impact rules and refine them over time. You can review how Pixee applies your preferences in the activity log for each repository.
</Tip>

When you first open the preferences editor, you'll see example content to help you get started. Replace it with rules tailored to your organization's standards and workflows.

## How Preferences Are Applied

Pixee uses a single source of preferences for each analysis run. The following precedence rules determine which source is used:

- **Repo-level `PIXEE.md`** takes priority when present. If a repository contains a `PIXEE.md` file, Pixee uses it and ignores organization preferences for that repository.
- **Organization preferences** serve as the baseline for any repository that does not have its own `PIXEE.md`.
- **Empty `PIXEE.md`** opts a repository out of organization preferences entirely. If you want a repository to have no preferences at all, commit an empty `PIXEE.md` file.

<Note>
There is no merging between sources. Pixee uses either the repo-level `PIXEE.md` or the organization preferences — never both. If you have requirements for merging preferences from multiple sources, please contact Pixee support.
</Note>

## Concurrent Editing

Organization preferences use optimistic locking to prevent silent overwrites. If another user saves changes while you are editing, you will see a conflict warning when you attempt to save. When this happens, refresh the page to load the latest version before making your changes again.
