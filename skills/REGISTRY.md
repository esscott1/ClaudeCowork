# Agent registry

One row per skill/agent in this repo. Maintained by Sally (the agent lifecycle manager) in the same commit as any agent change. "Installed" is the version saved as a skill in Eric's Claude account, which changes only when Eric reinstalls after a merge.

Stages: `proposed` → `draft` → `in-review` → `released` → `installed` → `deprecated` → `retired`.

| Agent | Category | Version | Stage | Surfaces | Schedule | Installed | Last change |
|---|---|---|---|---|---|---|---|
| job-lead-intake-scan | personal-productivity | 2.1.0 | in-review | cloud (scheduled) | daily 8:26am Mountain | pre-registry | 2026-10-09: score against Eric-Full-Resume.md (found by name) instead of Eric_Scott_Resume-Big2.docx |
| job-lead-tier2-scoring | personal-productivity | 2.1.0 | in-review | cloud (on demand or scheduled) | none | pre-registry | 2026-10-09: score against Eric-Full-Resume.md (found by name) instead of Eric_Scott_Resume-Big2.docx |
| job-lead-manual-jd-lookup | personal-productivity | 2.1.0 | in-review | desktop (Eric's browser) | none | pre-registry | 2026-10-09: score against Eric-Full-Resume.md (found by name) instead of Eric_Scott_Resume-Big2.docx |
| job-fit-rubric | personal-productivity | 1.1.0 | in-review | any (loaded by the three skills above) | none | not installed | 2026-10-09: score against Eric-Full-Resume.md (found by name) instead of Eric_Scott_Resume-Big2.docx |
| tailored-resume-builder | personal-productivity | 1.0.0 | in-review | cloud or desktop, on demand | none | not installed | 2026-10-09: new skill; fills the Job-Seeker-6 template from Eric-Full-Resume.md for a given job description |

"pre-registry" means the installed copy predates version tracking.
