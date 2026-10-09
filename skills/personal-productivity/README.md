# Personal productivity

Skills that automate parts of my own day-to-day workflow — starting with job search.

| Skill | Trigger | Depends on |
|---|---|---|
| [`job-lead-intake-scan`](job-lead-intake-scan/SKILL.md) | Runs on a daily schedule (8:26am Mountain) | Gmail, Google Drive and Google Sheets connectors only — cloud, no local computer needed |
| [`job-lead-tier2-scoring`](job-lead-tier2-scoring/SKILL.md) | On demand, or unattended on a schedule | Web search and fetch, Google Drive and Google Sheets connectors — cloud only, never LinkedIn |
| [`job-lead-manual-jd-lookup`](job-lead-manual-jd-lookup/SKILL.md) | On demand, only when I'm at my computer | My signed-in Chrome (LinkedIn) or the desktop app's built-in browser, plus the Sheets connector |
| [`job-fit-rubric`](job-fit-rubric/SKILL.md) | Never on its own; loaded by the three skills above whenever they set a Fit Score | Python 3 for `score.py` |
| [`tailored-resume-builder`](tailored-resume-builder/SKILL.md) | On demand, when I want a 2-page resume for a specific job | `Eric-Full-Resume.md` and the Job-Seeker-6 Word template in Drive, plus Python (`python-docx`, LibreOffice) |

The tracker is a Google Sheet edited in place through the Google Sheets connector, so every skill can update individual rows without replacing the file.

### One rubric, split into judgment and arithmetic

All three skills score fit the same way, through `job-fit-rubric`. The model reads the job description and the master resume and fills in an evidence table: each requirement quoted from the JD, marked required or preferred, and matched to the resume as strong, partial or none, plus title fit and work arrangement. `score.py` turns that table into the 1-10 score with fixed weights (required qualifications count three times as much as nice-to-haves) and hard ceilings (a job that requires relocating out of Colorado can't score above 3). The model's classifications stay visible and checkable, and the arithmetic is deterministic and unit-tested. The breakdown is written to each row's Notes, so every score shows how it was reached.

### Why separate skills

The stages have genuinely different requirements: intake and web-search scoring run unattended and must work even when my laptop is off, while the manual lookup needs a live, logged-in browser session and only makes sense when I'm at my computer. Splitting them means each skill's description names exactly when it applies, and none silently depends on something another doesn't need.

### The tiered-scoring idea

A week of job-alert email easily contains 40-80+ individual postings. Reading a full job description for every one of them isn't practical, so the pipeline screens cheap first (title/company/location from the alert email) and only escalates to a full job-description read (web search first, my own browser as the last resort) for leads worth the extra cost. Every logged row is tagged with which tier produced its score, so trust in the number is explicit rather than assumed.
