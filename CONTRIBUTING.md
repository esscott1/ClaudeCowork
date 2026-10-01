# How this repo is maintained

Changes are drafted through a chat with Claude, then landed here on their own feature branch and opened as a pull request rather than pushed straight to `main` — same review-before-merge discipline as any other codebase, even though the "codebase" is a set of skill instructions. `main` only moves when a PR is merged. Pull with `git pull` to sync your local clone whenever you want the latest merged state.

## Adding a new skill

1. Pick the category it belongs to under `skills/` (`personal-productivity`, `pm-enterprise`, `developer-tools`, `ai-governance`) — or propose a new one if it genuinely doesn't fit any of those.
2. Follow the conventions in [`docs/skill-template.md`](docs/skill-template.md) for the SKILL.md itself.
3. Add or update that category's `README.md` table with the new skill, trigger, and dependencies.
4. If the skill changes what's in the root `README.md`'s category index (a brand-new category, say), update that too.

## Working on `orchestration/`

Python code under `orchestration/job-search/` is a [uv](https://docs.astral.sh/uv/) workspace. Before opening a PR:

```bash
cd orchestration/job-search
uv sync --all-packages
uv run ruff check . && uv run ruff format --check .
uv run pytest -q
```

CI runs the same checks on every PR that touches `orchestration/job-search/`.

- **Business logic goes in `core/`; orchestration goes in `approaches/<name>/`.** If a graph edge or node starts making a business decision, move that rule into `core/policy.py`. The import-boundary test fails if `core` imports a framework.
- **Changing a prompt?** Bump its `version:` line and run the evals (`uv run python evals/job-fit/run_evals.py`). Commit the updated `results/latest.json` so the PR shows the before and after. The evals workflow re-runs them in CI when the `ANTHROPIC_API_KEY` secret is set.
- **Adding an orchestration approach?** Create `approaches/<name>/` as its own package that depends on `job-search-core`, give it a README stating its maturity (`spike` or `production`), and reuse the existing evals.
- **Never commit personal data.** Resumes, the tracker database, and OAuth tokens stay in the gitignored `data/` folder.
