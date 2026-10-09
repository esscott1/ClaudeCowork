# Agent registry

One row per agent in this repo. [Sally](ai-governance/sally/SKILL.md) updates this file in the same commit as any agent change; see her SKILL.md for what each column means and the lifecycle stages.

- **Version** is the version on `main`. **Installed version** is what Eric has in Claude, and stays `unverified` until someone compares the installed copy to the repo.
- **Install method** is `standalone` (the skill folder uploaded as a ZIP) or `plugin` (the category plugin installed from this marketplace). A skill is installed one way, never both.

| Agent | Category | Version | Stage | Install method | Installed version | Runs on | Schedule | Files | Last change |
|---|---|---|---|---|---|---|---|---|---|
| [`job-lead-intake-scan`](personal-productivity/job-lead-intake-scan/SKILL.md) | personal-productivity | 2.1.0 | in-review | standalone | unverified | Cloud (Gmail, Google Drive, Google Sheets) | Daily, 8:26am Mountain | SKILL.md only | 2026-10-09: tracker moved to Google Sheets, scored with job-fit-rubric against Eric-Full-Resume.md |
| [`job-lead-tier2-scoring`](personal-productivity/job-lead-tier2-scoring/SKILL.md) | personal-productivity | 2.1.0 | in-review | standalone | unverified | Cloud (web search and fetch, Google Drive, Google Sheets); no browser | On demand or scheduled | SKILL.md only | 2026-10-09: web-search JD lookup, hands unresolved rows to job-lead-manual-jd-lookup, scored against Eric-Full-Resume.md |
| [`job-lead-manual-jd-lookup`](personal-productivity/job-lead-manual-jd-lookup/SKILL.md) | personal-productivity | 2.1.0 | in-review | standalone | unverified | Eric's computer (signed-in browser) and Google Sheets | On demand | SKILL.md only | 2026-10-09: checked in from the installed copy; scored against Eric-Full-Resume.md |
| [`job-fit-rubric`](personal-productivity/job-fit-rubric/SKILL.md) | personal-productivity | 1.1.0 | in-review | not installed | — | Any surface; Python 3 for score.py | Loaded by the job-lead skills | SKILL.md, score.py, test_score.py | 2026-10-09: deterministic evidence-table scoring; evidence from Eric-Full-Resume.md |
| [`tailored-resume-builder`](personal-productivity/tailored-resume-builder/SKILL.md) | personal-productivity | 1.0.0 | in-review | not installed | — | Cloud or Eric's computer; python-docx and LibreOffice | On demand | SKILL.md, build_resume.py, test_build_resume.py | 2026-10-09: new; fills the Job-Seeker-6 template from Eric-Full-Resume.md for one job description |
| [`sally`](ai-governance/sally/SKILL.md) | ai-governance | 2.0.0 | released | standalone | 1.0.0 | Any surface; GitHub write access or the JobSearch Project queue | On demand | SKILL.md only | 2026-10-07: front door; status and drift moved to `sally-audit` |
| [`sally-audit`](ai-governance/sally-audit/SKILL.md) | ai-governance | 1.0.0 | released | not installed | — | Any surface; read-only | On demand | SKILL.md only | 2026-10-07: split out of `sally` as the read-only status and drift report |
