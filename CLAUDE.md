# CLAUDE.md

Guidance for Claude Code sessions in this repo. Claude reads this file at the start of every session.

## What this repo is

Eric's public portfolio of agentic work, meant for prospective employers in AI enablement consulting. It has two halves:

- `skills/`: Claude skills (SKILL.md files) grouped by category. These are working copies of skills that are installed and run in the Claude app.
- `orchestration/job-search/`: the same job-search workflow as an owned, code-defined pipeline (LangGraph). See its README and `docs/adr/`.

Treat everything here as public. Code quality, commit messages and PR descriptions are part of the portfolio.

## Git workflow

- **Never commit to `main`.** Make every change on a feature branch (`feat/…`, `fix/…`, `docs/…`, `restructure/…`) and open a pull request. Eric reviews and merges; Claude does not merge.
- Keep each PR to one concern. The description says what changed, why, what was verified, and what wasn't.
- **"Clean up"** means: for each remote branch other than `main`:
  1. Confirm its PR is **merged**.
  2. Confirm the branch has **no commits missing from `main`**, for example with `git log origin/main..origin/<branch>` returning nothing.
  3. Delete the branch on the remote (`git push origin --delete <branch>`) and locally, then `git fetch --prune`.

  List what will be deleted before deleting it. Never delete `main`, an unmerged branch, or a branch whose PR is still open. Do this only when Eric asks to clean up. Automatic deletion on merge is deliberately **off** for this repo.
- If the session can't delete branches (cloud sessions push only to their own branch), do the checks anyway and give Eric the exact command to run.

## Skills (`skills/`)

- Follow `docs/skill-template.md`. Front-load the `description` with when the skill runs (scheduled, on demand, needs the computer).
- When you add or change a skill, update that category's `README.md` table.
- **Editing a SKILL.md here does not update the installed skill** in Eric's Claude account. Installing is a separate step in the Claude app. Say so in the PR when a change needs reinstalling. The repo and the installed skills can drift, so check before assuming they match.

## Orchestration (`orchestration/job-search/`)

This is a uv workspace with one framework-free package (`core/`) and one package per orchestration approach (`approaches/<name>/`).

- **Business logic goes in `core`; orchestration goes in `approaches/`.** `core` must never import LangGraph, LangChain, or any other orchestration framework; `core/tests/test_import_boundary.py` enforces this.
- **Routing decisions are business rules.** They live in `core/policy.py`. Graph edges call them and never re-implement them.
- **Status changes only go through `tracker.transition()`**, which enforces `domain.TRANSITIONS`. Don't bypass it.
- **Nodes stay thin:** call a `core` step, record the result with a `core.usecases` function, return a small state update. A node that calls `interrupt()` does nothing with side effects before the interrupt, because LangGraph re-runs the node on resume.
- **A new orchestration approach** is a new folder under `approaches/` that depends on `job-search-core`, with a README stating its maturity (`spike` or `production`). Nothing in `core` should need to move.
- **Changing a prompt** (`core/src/job_search_core/prompts/`): bump its `version:` line, run the evals, and commit `evals/job-fit/results/latest.json` so the PR shows the effect.

Before opening a PR that touches this folder:

```bash
cd orchestration/job-search
uv sync --all-packages
uv run ruff check . && uv run ruff format --check .
uv run pytest -q
```

Use the same commands CI does. Tests use fakes only, so they need no keys or network.

## Personal data: never commit it

The real resume, the tracker database, LangGraph checkpoints, OAuth credentials and tokens, and `.env` all stay in the gitignored `data/` folder or in `.env`. Evals and demos use the synthetic persona in `evals/job-fit/`. Before committing, check that `git status` shows nothing under any `data/` folder except `data/README.md`.

## Where things run

- **Local (VS Code / terminal):** building and testing the pipeline, `jobs auth google` (opens a browser), real evaluations against `data/`, LangGraph Studio, and all git operations.
- **Claude app:** the currently scheduled skills and their Gmail/Drive connectors, and installing skills into Eric's account.

The setup guide is `docs/setup/job-search-langgraph.md`.
