---
name: "job-lead-manual-jd-lookup"
description: "On-demand, requires Eric's computer: for tracker rows job-lead-tier2-scoring flagged as unreachable by web search (LinkedIn postings with no JD body, sites that block fetch), read the real JD using Eric's own browser session and update the row."
---

# Job Lead Manual JD Lookup

The manual-fallback half of the Tier 2 pipeline. `job-lead-tier2-scoring` runs cloud-only (web search + WebFetch, no browser) and handles most rows; anything it can't get a JD for - most commonly a LinkedIn posting with "Promoted by hirer / Responses managed off LinkedIn" and no JD body, or a site that blocks WebFetch entirely (JS-rendered career page, 403, a mirror site that needs interactive access) - it flags in the row's Notes rather than guessing. This skill picks up exactly those flagged rows and uses Eric's own logged-in browser to get the real text. It exists as its own skill (rather than a fallback branch inside job-lead-tier2-scoring) specifically so Eric can run it himself, on demand, whenever he's at his machine - it must never run unattended, since it depends on a live browser session and, for some sites, one-time interactive site-access approval that nobody is present to grant on a schedule.

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
- Otherwise, read the tracker and find every row whose Notes contains a job-lead-tier2-scoring recommendation to run this skill (look for "job-lead-manual-jd-lookup" or "Tier 2 (web search) attempted" in Notes) and that hasn't already been upgraded to "Full JD read".
- If there are a lot, do the first few and check in before continuing - this is a slower, browser-driven process per row.

## 2. Fetching the real JD
- **LinkedIn job-view links** (`linkedin.com/.../jobs/view/<id>/`): use **Claude in Chrome**, which runs in Eric's real, already-logged-in Chrome (load its tools via ToolSearch first if deferred). Navigate to the URL, then `get_page_text`. The JD body often loads asynchronously or sits behind a "see more" expansion - wait ~2s and retry once. If it's still not there, the listing genuinely has no JD on LinkedIn (common on "Promoted by hirer / Responses managed off LinkedIn" postings) - leave the row's score as-is, note that LinkedIn confirmed no JD, and move on rather than looping.
- **Other blocked sites** (a mirror or career page job-lead-tier2-scoring found but couldn't fetch - 403, JS-rendered, or needs a login): use the **built-in browser**. It will likely need one-time site access - call `request_access` for that site (scope "site" is fine so it doesn't ask again, or "once" if Eric would rather approve each time) and retry. If Eric declines or doesn't respond to an access prompt, don't push - note that the site couldn't be accessed and move to the next row.
- Start from the candidate URLs job-lead-tier2-scoring already tried (check the row's Job Description link and Notes) before searching fresh - no need to redo its web search from scratch unless those leads are dead ends too.

## 3. Scoring
Load the `job-fit-rubric` skill and follow it, exactly as `job-lead-tier2-scoring` does: build the `tier2` evidence table from the JD you read, run its `score.py`, and use the returned `score` unchanged as the Fit Score. Jobs requiring relocation out of Colorado are capped at 3. Write the 3 biggest gaps into the single Gaps cell in the short bulleted format defined in `job-fit-rubric`.

## 4. Updating the row
- Update the EXISTING row in place - never duplicate. Never touch Status.
- Write updates with the Sheets connector, re-finding each row by ID just before writing it.
- If a usable JD was found: overwrite Fit Score and Gaps, set Notes to `"Full JD read (job-lead-manual-jd-lookup, browser) - <LinkedIn direct / site name>. <breakdown from score.py>"`, and set Last Updated to today.
- If the posting genuinely has no JD anywhere reachable (LinkedIn confirmed empty and no other source exists), leave the score as-is but update Notes to say so plainly (e.g. `"Manual lookup attempted <date> - LinkedIn confirmed no JD body, no other source found. Still Tier 1."`) and Last Updated to today, so it doesn't get re-flagged forever.

## 5. End-of-run summary
List which rows were upgraded with their new scores, which still couldn't be reached (and why), and how many are still pending if this was a partial sweep.

## Known constraints
- Requires Eric's computer on and reachable: Chrome running with the extension for LinkedIn rows, and/or the desktop app's built-in browser for other blocked sites. If neither is reachable, say so plainly and stop rather than silently falling back to WebFetch (which is exactly what already failed upstream in job-lead-tier2-scoring).
- Never run this on a schedule - it depends on live interactive access (browser session, and possibly a site-access approval) that nobody is present to grant unattended.
- Never send email, reply to a lead, or click apply.
- Never advance a row's Status - that stays Eric's call.