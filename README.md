# ClaudeCowork

A portfolio of Claude Cowork skills I'm building and iterating on — kept here to document the pattern, not just the output. Skills span a deliberately broad range: personal productivity automations, enterprise PM/reporting tooling, developer-focused code tasks, and AI governance/guardrail checks.

> **JobSearchAssistant: a Claude Cowork plugin.** The job-search skills in this repo install together as one plugin that scans email for new job leads, screens them for fit, and logs them to a tracker. See [Install as a plugin](#install-as-a-plugin).

## Categories

| Category | What it's for |
|---|---|
| [`skills/personal-productivity/`](skills/personal-productivity/README.md) | My own day-to-day workflow automation — currently a job-search lead intake and scoring pipeline |
| [`skills/pm-enterprise/`](skills/pm-enterprise/README.md) | Project/program manager tooling — status dashboards built from Jira, Confluence, RAG-tracked docs, etc. |
| [`skills/developer-tools/`](skills/developer-tools/README.md) | Engineering workflow skills — unit testing, linting, and other code-quality tasks |
| [`skills/ai-governance/`](skills/ai-governance/README.md) | Gatekeeper-style skills — guardrail and policy checks for AI agent activity |

Each category has its own README with the skills currently published there, their triggers, and dependencies.

## Install as a plugin

This repo is a Claude plugin marketplace ([`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json)). Each category that has skills is one installable plugin.

- **claude.ai or the desktop app:** Customize > Plugins, add `esscott1/ClaudeCowork` as a marketplace, then install a plugin from it.
- **Claude Code:** `claude plugin marketplace add esscott1/ClaudeCowork`, then `claude plugin install personal-productivity@esscott-claudecowork`.

| Plugin | What it is | Skills |
|---|---|---|
| `personal-productivity` | JobSearchAssistant, a Cowork plugin for job-lead intake and fit scoring | `job-lead-intake-scan`, `job-lead-tier2-scoring` |
| `ai-governance` | Sally, the lifecycle manager that lands every skill change here as a reviewed PR, and her read-only status and drift report | `sally`, `sally-audit` |

## Orchestration: the same workflow, owned end to end

Skills are one way to run a workflow. [`orchestration/`](orchestration/job-search/README.md) holds the other: code-defined pipelines where I own the integrations, state and failure handling.

| Path | What it is |
|---|---|
| [`orchestration/job-search/`](orchestration/job-search/README.md) | The job-search workflow as a LangGraph state graph. Business logic lives in a framework-free `core` package, and each orchestration approach is a sibling under `approaches/` |
| [`docs/adr/`](docs/adr/) | Architecture decision records: why LangGraph here, why the tracker is the system of record |
| [`docs/setup/`](docs/setup/job-search-langgraph.md) | Setup guide, including Google OAuth for a personal account |

The skills version keeps running while the orchestrated version runs alongside it in shadow mode. Each part is cut over once its evals match ([ADR 0001](docs/adr/0001-orchestration-approach.md)).

## Conventions

Every SKILL.md here follows a common shape, documented in [`docs/skill-template.md`](docs/skill-template.md), so skills stay easy to compare across categories as the list grows.

## Status

Actively iterating — these are working copies of skills I run against my own work, published here to document the pattern as I refine it.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for how this repo is maintained (branch + PR workflow) and how to add a new skill.
