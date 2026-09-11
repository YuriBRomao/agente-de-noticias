---
description: Create a weekday issue summarizing all open issues and pull requests grouped by label.
on:
  schedule: daily on weekdays
  workflow_dispatch:
permissions:
  contents: read
  issues: read
  pull-requests: read
tools:
  github:
    mode: gh-proxy
    toolsets: [default]
safe-outputs:
  create-issue:
    max: 1
    title-prefix: "Daily Digest –"
---

# Daily Digest

Create the repository's daily open-work digest as one GitHub issue.

## Data collection

1. Determine today's date in UTC as `YYYY-MM-DD`.
2. Read every open issue and every open pull request in the current repository. Do not limit the result to recently updated items. Use pagination or a sufficiently high limit so no open item is omitted.
3. For each item, collect its number, URL, title, author, labels, creation timestamp, and type (`Issue` or `Pull Request`).
4. Calculate how long each item has been open from its creation timestamp through the workflow start time in UTC, using a concise human-readable duration such as `3d 4h`.

## Report

Create one issue using the configured `create-issue` safe output. Pass only today's UTC date (`YYYY-MM-DD`) as the safe output title; its configured prefix will make the final issue title exactly `Daily Digest – <date>`.

In the body:

- Start with the total number of open items, counting each issue or pull request once.
- Group the complete item list by label. Include an item in every label group it has; use `Unlabeled` for items with no labels. Do not invent labels.
- Under each group, list every item with its type, number, linked title, author, and time open.
- Include a short count for each label group and a final count split between issues and pull requests.
- Keep the title, author, URL, and duration visible for every item; do not hide the required item details inside a collapsed section.

Use `noop` with a short reason only when an issue with the exact title `Daily Digest – <date>` already exists, to prevent duplicate digests from a manual rerun on the same day. A repository with zero open issues and pull requests still requires a digest issue reporting total `0`.
