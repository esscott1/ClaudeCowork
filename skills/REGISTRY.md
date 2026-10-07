# Agent registry

One row per agent in this repo. [Sally](ai-governance/sally/SKILL.md) updates this file in the same commit as any agent change; see her SKILL.md for what each column means and the lifecycle stages.

- **Version** is the version on `main`. **Installed version** is what Eric has in Claude, and stays `unverified` until someone compares the installed copy to the repo.
- **Install method** is `standalone` (the SKILL.md saved as a skill) or `plugin` (the category plugin installed from this marketplace). A skill is installed one way, never both.

| Agent | Category | Version | Stage | Install method | Installed version | Runs on | Schedule | Last change |
|---|---|---|---|---|---|---|---|---|
| [`job-lead-intake-scan`](personal-productivity/job-lead-intake-scan/SKILL.md) | personal-productivity | 1.0.0 | released | standalone | unverified | Cloud (Gmail + Google Drive) | Daily, 8:26am Mountain | 2026-09-29: moved into the `personal-productivity` category |
| [`job-lead-tier2-scoring`](personal-productivity/job-lead-tier2-scoring/SKILL.md) | personal-productivity | 1.0.0 | released | standalone | unverified | Eric's computer (signed-in browser) | On demand | 2026-09-29: moved into the `personal-productivity` category |
| [`sally`](ai-governance/sally/SKILL.md) | ai-governance | 1.1.0 | released | standalone | 1.0.0 | Any surface; GitHub write access or the JobSearch Project queue | On demand | 2026-10-07: plugin marketplace awareness, registry, both install methods |
