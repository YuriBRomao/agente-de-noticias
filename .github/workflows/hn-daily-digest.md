---
description: Create a weekday Hacker News digest for enterprise software developers.
on:
  schedule: daily on weekdays
  workflow_dispatch:
permissions:
  contents: read
  issues: read
  pull-requests: read
  copilot-requests: write
engine:
  id: copilot
  model: gpt-5
tools:
  github:
    mode: gh-proxy
    toolsets: [default]
network: defaults
safe-outputs:
  create-issue:
    max: 1
    title-prefix: "HN Digest –"
---

# HN Daily Digest

Create one daily Hacker News digest issue for professional developers working in large companies.

## Collect and filter stories

1. Determine today's date in UTC as `YYYY-MM-DD`.
2. Fetch the top 30 story IDs from `https://hacker-news.firebaseio.com/v0/topstories.json`.
3. Fetch each story's details from `https://hacker-news.firebaseio.com/v0/item/<id>.json`. Continue past unavailable or deleted items, and do not substitute stories outside the first 30 IDs.
4. Keep only stories with a score strictly greater than 100 and a clear primary topic in at least one of these categories: software engineering, cloud infrastructure, AI/ML, developer tooling, or distributed systems. Do not include general business, politics, finance, cryptocurrency, or unrelated consumer technology stories.
5. For every qualifying story, collect its title, URL, score, number of comments (`descendants`, or `0` when absent), and a factual one-sentence explanation of why it is relevant to enterprise developers. Use the Hacker News item URL when the story has no external URL.

## Create the issue

Before writing, search the current repository for an existing issue with the exact final title `HN Digest – <date>`. If one exists, call `noop` with a short duplicate explanation and do not create another issue.

Otherwise, use the configured `create-issue` safe output exactly once. Pass only today's UTC date (`YYYY-MM-DD`) as the title so the configured prefix produces the exact final title `HN Digest – <date>`.

The issue body must contain:

- A brief statement of the date and that the source was the top 30 Hacker News stories.
- The total number of qualifying stories.
- A Markdown table with one row per qualifying story and these columns: `Title`, `URL`, `Score`, `Comments`, and `Enterprise relevance`.
- Valid Markdown links for every story title or URL. Escape pipe characters in titles or summaries so the table remains valid.
- If no stories qualify, still create the issue and state that no stories met the score and topic criteria.
