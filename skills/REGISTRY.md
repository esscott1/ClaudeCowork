# Agent registry

One row per skill/agent in this repo. Maintained by Sally (the agent lifecycle manager) in the same commit as any agent change. "Installed" is the version saved as a skill in Eric's Claude account, which changes only when Eric reinstalls after a merge.

Stages: `proposed` → `draft` → `in-review` → `released` → `installed` → `deprecated` → `retired`.

| Agent | Category | Version | Stage | Surfaces | Schedule | Installed | Last change |
|---|---|---|---|---|---|---|---|
| job-lead-intake-scan | personal-productivity | 2.0.0 | in-review | cloud (scheduled) | daily 8:26am Mountain | pre-registry | 2026-10-03: tracker moved to Google Sheets; Fit Score from job-fit-rubric |
| job-lead-tier2-scoring | personal-productivity | 2.0.0 | in-review | cloud (on demand or scheduled) | none | pre-registry | 2026-10-03: tracker moved to Google Sheets; scoring split into evidence table + score.py |
| job-lead-manual-jd-lookup | personal-productivity | 2.0.0 | in-review | desktop (Eric's browser) | none | pre-registry | 2026-10-03: added to repo; tracker moved to Google Sheets; scoring via job-fit-rubric |
| job-fit-rubric | personal-productivity | 1.0.0 | in-review | any (loaded by the three skills above) | none | not installed | 2026-10-03: created; rubric v1.0 |

"pre-registry" means the installed copy predates version tracking.
