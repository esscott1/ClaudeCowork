---
name: "job-lead-tier2-scoring"
description: "Cloud-only, cron-friendly: for tracker rows still tagged Tier 1, search the open web and fetch the real JD from company/ATS pages (never LinkedIn directly), then score and update the row. Rows it can't resolve get flagged in Notes recommending job-lead-manual-jd-lookup."
---

# Job Lead Tier 2 Scoring (web-search based)

The enrichment half of a two-skill pipeline. `job-lead-intake-scan` runs daily and logs new leads with a quick Tier 1 (title-only) score; this skill takes rows still tagged Tier 1, tries to find the *real* job description via open web search, and rewrites the row with a properly-sourced score and gaps. It never opens LinkedIn itself and never touches a browser or the linked device, so it can run entirely unattended (a scheduled task) or on demand. A separate skill, `job-lead-manual-jd-lookup`, exists for the rows this skill can't resolve (LinkedIn postings with no JD body, sites that block fetch/search) and requires Eric's own computer - it is run by Eric, not called automatically from here.

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


## 1. Figure out which rows to work on
- If Eric named specific row IDs or companies, use those.
- If he asked for "the LinkedIn rows" or similar, read the tracker and find every row whose Notes says "Tier 1 screen" (or equivalent) and has no "Full JD read" tag yet.
- On a scheduled run, or a general sweep, do all outstanding Tier-1 rows that don't already carry a "Tier 2 (web search) attempted" note from today (dedup against same-day re-runs), but check in (or, on schedule, just note it in the end-of-run summary) if there turn out to be a lot - each row costs a few search+fetch calls.
- Skip any row already tagged "Full JD read" - don't re-score it.

## 2. Finding the real JD - web search only, no browser
For each row:
1. Build a search query from Company + exact Job Title, quoted (e.g. `"Acme Corp" "Senior Widget Architect" job description`). Add a distinguishing detail (location, req/job number if visible in the LinkedIn URL slug or the row) if the title alone is generic.
2. Prefer, in this order: (a) the company's own careers page, (b) a direct ATS posting (`job-boards.greenhouse.io`, `jobs.lever.co`, iCIMS, Workday, `careers.<company>.com`), (c) a job-board mirror (Indeed, BuiltIn, ZipRecruiter, dreamworkhq, freehire, Monster, Dice, TheMuse, JobRight, Lensa, CareerBuilder, etc.) only if (a)/(b) aren't found or don't fetch cleanly.
3. WebFetch the most promising 2-3 candidate URLs, asking for responsibilities, required qualifications, and preferred qualifications verbatim. If one comes back empty (JS-rendered SPA with no server-rendered text), a 403, or says the posting is closed/filled, move to the next candidate rather than retrying the same URL.
4. **Never fetch a `linkedin.com/jobs/view/...` URL directly** - WebFetch is blocked by LinkedIn's robots.txt regardless of how the search led there. If every candidate outside LinkedIn fails, that's this skill's dead end for the row (see step 4).
5. Stop after 2-3 solid attempts per row - this is meant to be a fast automated pass, not an exhaustive hunt.

## 3. Scoring
Load the `job-fit-rubric` skill and follow it. In short:
1. **Judgment:** build its evidence table in `tier2` mode: every requirement quoted from the JD, marked required or preferred, with Eric-Full-Resume.md evidence strong / partial / none, plus title/seniority and work arrangement (remote, Colorado on-site or hybrid, relocation required, or unknown).
2. **Arithmetic:** run its `score.py` on the table. The `score` it returns is the Fit Score, unchanged. If it looks wrong, fix the table, not the number.

Eric won't relocate from Colorado; `score.py` caps any job requiring relocation at 3.

Write the 3 biggest gaps into the single Gaps cell in the short bulleted format defined in `job-fit-rubric`.

## 4. Updating the row
- Update the EXISTING row in place - never create a duplicate. Never touch Status - leave whatever Eric has set it to.
- Do the research for all rows first, then write the updates row by row with the Sheets connector, re-finding each row by ID just before writing it.
- If the JD lists pay and the row's Pay Range is blank, fill it in. Put the source URL in Notes so job-lead-manual-jd-lookup and Eric can find it.
- If the posting turns out to be closed or filled, say so in Notes and suggest Eric set Status to Closed (don't set it yourself); don't recommend a manual lookup for it.
- **If a usable JD was found:** overwrite Fit Score and Gaps, set Notes to `"Full JD read (job-lead-tier2-scoring, web search) - JD from <domain of the source URL>. <breakdown from score.py>"`, and set Last Updated to today.
- **If no usable JD could be found after trying the candidates in step 2:** do NOT touch Fit Score or Gaps (leave the existing Tier 1 score standing). Instead update Notes to record the attempt and point at the manual fallback, e.g.: `"Tier 2 (web search) attempted <date> - no fetchable JD found off LinkedIn ([reason: e.g. no non-LinkedIn source found / sources blocked or JS-rendered / posting appears filled]). Tried: <domains tried>. Recommend running job-lead-manual-jd-lookup (needs Eric's browser) to check the LinkedIn posting or blocked sites directly."` Still update Last Updated to today so Eric can see this was recently touched.

## 5. End-of-run summary
Always end with a short summary, never silence:
- Rows upgraded to a full JD read, with their new scores.
- Rows flagged for manual lookup, with why (one line each).
- How many are still pending if this was a partial sweep.

## Known constraints
- Cloud-only by design: never use a device/browser tool here, and never attempt LinkedIn directly - that's `job-lead-manual-jd-lookup`'s job.
- Never send email, reply to a lead, or click apply - read-only research.
- Never advance a row's Status - that stays Eric's call.
- Never set a Fit Score that didn't come from `job-fit-rubric`'s `score.py`.
- Safe to run unattended on a schedule since every tool it uses (WebSearch, WebFetch, the Drive and Sheets connectors) works without Eric present or any interactive approval.