---
name: "job-lead-intake-scan"
description: "Scan Eric's Gmail for new job leads (Route36/Charlie, LinkedIn job alerts, MyJobHelper), screen them for fit, and log new candidates to the Job Search Tracker. Cloud-only — runs on the daily schedule, no computer/browser needed."
---

# Job Lead Intake Scan

Finds new job leads in Eric Scott's inbox, does a first-pass fit screen, and logs them to his Job Search Tracker. This is the intake stage of a three-skill pipeline: this skill only ever writes rows with status "Need to review" and a Tier tag in Notes — it never advances a status, never emails anyone, never generates a resume, and never opens a LinkedIn job page. Real-JD scoring happens later in `job-lead-tier2-scoring` (cloud, web search, can run unattended), and anything it can't reach goes to `job-lead-manual-jd-lookup` (Eric's own browser, on demand).

Runs daily via a scheduled task (8:26am Mountain) entirely through the Gmail, Google Drive and Google Sheets connectors — by design it must work even when Eric's laptop is off, so never reach for a device or browser tool here.

## Key files and systems
- Folder: Google Drive "JobSearch2026", folder id `1rEJr3f7alZN0CPpk7qesZwOE0G2bFCfd`.
- Career inventory (the only evidence used for scoring): `Eric-Full-Resume.md`, in the folder above. Find it by exact name (`title = 'Eric-Full-Resume.md'` and the folder's `parentId`) rather than a remembered file ID, and read it with `read_file_content`. It holds all of Eric's experience, not a two-page selection, so score against the whole file. Read-only for these skills; Eric edits it himself. If it isn't found, stop and report that; don't fall back to the old Big2 .docx, which is superseded.
- Tracker: the Google Sheet `Job_Search_Tracker` in JobSearch2026, tabs "Job Log" and "Status Guide". Read and write it with the **Google Sheets connector**, which edits cells in place, so its file ID and link never change.
- Scoring: the `job-fit-rubric` skill. Load it with the Skill tool before setting any Fit Score; it holds the evidence-table format and `score.py`.

## Finding the tracker
Call the Drive connector's `search_files` with:
`title = 'Job_Search_Tracker' and mimeType = 'application/vnd.google-apps.spreadsheet' and parentId = '1rEJr3f7alZN0CPpk7qesZwOE0G2bFCfd'`
- Exactly one result: its `id` is the `spreadsheetId` for every Sheets call.
- No result: stop and report it. Never create a new tracker.
- The old Excel file (`Job_Search_Tracker (archived xlsx ...).xlsx`) is a read-only archive from before the move to Google Sheets. Never read rows from it or write to it.

## Reading and writing the tracker
- **Read** with `get_values` on `'Job Log'!A1:Z`. Row 1 is the header. Find each column by its header text on every run rather than assuming a column letter, so a column Eric adds or moves doesn't break anything.
- **Find rows by ID** (`JS-0xx` in the "ID" column) at the moment you write, never by a row number remembered from earlier. Eric may sort or filter the sheet between your read and your write. Re-read the row just before writing and confirm its ID matches.
- **Update** with `update_values` on just the cells this skill owns in that row (A1 range like `'Job Log'!J14:M14`). Don't rewrite whole rows or the whole sheet.
- **Links:** write the Job Description cell with `update_formulas` as `=HYPERLINK("<url>","View posting")`.
- **Counts** on the "Status Guide" tab are `COUNTIF` formulas. Don't write numbers over them.
- IMPORTANT: if a Google Sheets call fails because the connector isn't available or authorized, stop and report "Google Sheets connector unavailable". Don't fall back to downloading or uploading an .xlsx, or to editing a copy on Eric's computer.

### Adding rows
Find the last row with a real `JS-0xx` value in the ID column. Insert the new rows directly after it with `insert_dimension` (`dimension` ROWS, `inheritFromBefore` true, so they pick up that row's formatting and dropdowns), then fill them with `update_values`. Never append below empty formatted rows, where Eric won't see them. Write all of a run's new rows in one pass.

- Route36 leads/resumes subfolder: `JobSearch2026/Route36`

## 1. Look back 7 days, always
Always search a trailing 7-day window (`newer_than:7d`), regardless of how often this skill is actually scheduled to run — if the cron interval is ever widened, a fixed 7-day lookback still won't miss anything, it'll just reprocess a few already-seen emails, which dedup (below) handles for free. Do not narrow the window to match the schedule.

## 2. Gmail sources to scan
- Route36 / Charlie: `from:charlie@r36.com`
- LinkedIn single-job alerts: `from:jobalerts-noreply@linkedin.com` — each email is itself a digest of 2-6 job cards (title/company/location, sometimes a salary band), not one job. Parse every card, not just the one in the subject line.
- LinkedIn "similar jobs" recommendations: `from:jobs-noreply@linkedin.com` — lower-signal digests; include but treat as lower priority.
- MyJobHelper: `from:info@alerts.myjobhelper.com` — single-posting alerts, but low-signal/high-noise (many are generic mismatched blasts, e.g. retail "Full Time Assistant"). Screen hard.

Explicitly EXCLUDE other LinkedIn mail that isn't a job posting: `messages-noreply@`, `notifications-noreply@`, `groups-noreply@`, `updates-noreply@`, `editors-noreply@linkedin.com`.

If Eric mentions a new source, add its sender pattern here.

## 3. Screening and scoring (Tier 1, plus opportunistic real fetches)
From the email body alone (title, company, location, salary if shown), judge plausible fit against Eric's background: enterprise/solutions architecture, IT PMO/delivery leadership, technical program/project management, AI enablement and adoption strategy, cloud/infrastructure consulting. Discard obvious non-fits (individual-contributor retail/trades roles, unrelated disciplines) without adding them to the tracker. When unsure, err toward including it.

- **LinkedIn and MyJobHelper leads:** score from the alert content only (this is "Tier 1" — title/company-based, not a real JD read). Never try to open the posting here — LinkedIn blocks WebFetch via robots.txt and requires a signed-in browser this skill doesn't have. Finding the real JD is `job-lead-tier2-scoring`'s job (web search off LinkedIn), with `job-lead-manual-jd-lookup` as the browser fallback.
- **Route36 links and other non-LinkedIn ATS links** (Greenhouse, Lever, iCIMS, Workday, direct company career pages): these don't need Eric's own browser session, so you may attempt a real fetch with WebFetch and score those with a full JD read (if the site blocks WebFetch, leave the row at Tier 1 - `job-lead-tier2-scoring` or `job-lead-manual-jd-lookup` will pick it up) — tag those rows as fully-scored rather than Tier 1.

**Setting the Fit Score.** Every score comes from the `job-fit-rubric` skill; never estimate one by feel.
- Tier 1 rows: build the evidence table in `tier1` mode (title/seniority and work arrangement from the alert email) and run its `score.py`.
- Full-JD rows: build the `tier2` table from the fetched JD and run `score.py`.
- Work arrangement matters even at Tier 1: Eric won't relocate from Colorado, and `score.py` caps any job that requires relocation at 3. Log those rows anyway (don't discard them) so Eric can see them; the low score keeps them out of his way.

## 4. Dedup before writing
Before adding any row, read the current tracker's Company + Job Title (and Job Description link, if present) and skip anything already logged. Match loosely (same company + substantially same title) since the same LinkedIn job can arrive in more than one digest with different tracking-parameter URLs.

## 5. Writing new rows
- Status is always **"Need to review"**.
- Continue the existing `JS-0xx` ID sequence.
- Fill: Date Found (today), Source, Company, Job Title, Job Description (hyperlink to the posting, or the Gmail message if there's no direct link), Type/Location, Pay Range (if stated), Fit Score, Gaps (format per `job-fit-rubric`), Status, Last Updated. Leave Tailored Resume / Resume Created / Date Submitted / Submission Email blank.
- In Next Action/Notes, tag the row "Tier 1 screen" or "full JD" so Eric (and `job-lead-tier2-scoring`) know how much to trust the score, followed by the `breakdown` string `score.py` returned. The tracker's "Status Guide" tab documents the tiers and the rubric; keep that legend in sync if the logic changes.

## 6. End-of-run summary
Always end with a short summary, never silence: how many new rows were added (one-line list), how many were screened out at Tier 1 and roughly why, and anything flagged as unreachable. If nothing new was found, say so explicitly.

## Known constraints
- Never mark anything "Good fit", "Submitted", or advance a status — this skill only proposes.
- Never send email, reply to a lead, or click apply — read-only scan.
- Never use a device/browser tool here — that dependency belongs entirely to `job-lead-manual-jd-lookup`.
- Never set a Fit Score that didn't come from `job-fit-rubric`'s `score.py`.
- Gmail attachments (a JD sent as a raw attachment, not a link) can't be downloaded directly — flag it in Notes for Eric to "Add to Drive" himself.