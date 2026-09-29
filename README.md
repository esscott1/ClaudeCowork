# ClaudeCowork

Cowork skills I'm building and iterating on for my job search automation, kept here as a portfolio of the pattern — not just the output.

## What's here

A small pipeline of Claude skills that work together to find, screen, and score job leads from my inbox, and log them to a tracking spreadsheet:

| Skill | Trigger | Depends on |
|---|---|---|
| [`job-lead-intake-scan`](skills/job-lead-intake-scan/SKILL.md) | Runs on a daily schedule (8:26am Mountain) | Gmail + Google Drive only — cloud, no local computer needed |
| [`job-lead-tier2-scoring`](skills/job-lead-tier2-scoring/SKILL.md) | Run on demand | A signed-in browser session on my own computer (LinkedIn blocks unauthenticated fetches) |

### Why two skills instead of one

The two halves have genuinely different requirements: intake runs unattended on a fixed schedule and must work even when my laptop is off, while scoring needs a live, logged-in browser session and only makes sense when I'm at my computer to kick it off or review the result. Splitting them means each skill's description names exactly when it applies, and neither one silently depends on something the other doesn't need.

### The tiered-scoring idea

A week of job-alert email easily contains 40-80+ individual postings. Reading a full job description for every one of them isn't practical, so the pipeline screens cheap first (title/company/location from the alert email) and only escalates to a full job-description read — which requires a real browser session — for leads worth the extra cost. Every logged row is tagged with which tier produced its score, so trust in the number is explicit rather than assumed.

## Status

Actively iterating — these are working copies of skills I run against my own job search, published here to document the pattern as I refine it.

## How this repo is maintained

Changes are drafted through a chat with Claude, then landed here on their own feature branch and opened as a pull request rather than pushed straight to `main` — same review-before-merge discipline as any other codebase, even though the "codebase" is a set of skill instructions. `main` only moves when a PR is merged. Pull with `git pull` to sync your local clone whenever you want the latest merged state.
