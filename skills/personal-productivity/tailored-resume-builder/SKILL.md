---
name: "tailored-resume-builder"
description: "On demand, when Eric asks for a tailored resume for a specific job (he supplies or points to the job description): picks the most relevant true content from Eric-Full-Resume.md and fills the Job-Seeker-6 Word template into a 2-page .docx and PDF. Never invents experience; reports gaps instead. Not for scoring (that's job-fit-rubric) and not for editing the master inventory."
---

# Tailored Resume Builder (v1.0)

Turns one job description into one 2-page resume in Eric's Job-Seeker-6 layout. The model does the judgment (which true things to show, in what words); `build_resume.py` does the layout, so every resume has the same format as the template instead of a hand-built approximation.

## Key files and systems
- Folder: Google Drive "JobSearch2026", folder id `1rEJr3f7alZN0CPpk7qesZwOE0G2bFCfd` (on Eric's computer, the synced copy of it).
- Content source (the only source of facts): `Eric-Full-Resume.md`. Find it by exact name (`title = 'Eric-Full-Resume.md'` and the folder's `parentId`), never by remembered file ID. Read-only here; Eric edits it himself. If it isn't found, stop and say so.
- Layout source (format only, no content): `Job-Seeker-6-Resume-Template-.docx`, found by exact name in the same folder. The template's sample text (a different person's career) must never reach the output; the script replaces all of it.
- `build_resume.py` (this directory): `python3 build_resume.py --template <template.docx> --content content.json --out <file.docx> --pdf`. Needs `python-docx`; page counting needs LibreOffice (`soffice`) and `pdfinfo`. It prints JSON with `pages` and `warnings`, and exits 2 with a reason if `content.json` is invalid.
- `evals/cases.md`: starter eval cases, with a synthetic fixture in `evals/fixtures/`.

## Procedure
1. **Get the job description.** Eric attaches it, pastes it, or names a tracker row (then find the JD source URL in Notes). If there is no usable JD, stop and ask. Never tailor to a job title alone.
2. **Read both files** (content source, template). Download the template to a scratch directory; don't edit it in place.
3. **Map the job to the inventory.** Load `job-fit-rubric` and build its evidence table (each JD requirement quoted, marked required or preferred, matched strong/partial/none to a pointer in `Eric-Full-Resume.md`). Use it as the selection guide: strong matches get the most room, `none` matches are gaps.
4. **Write `content.json`** (shape below) using only what the inventory supports:
   - Keep every employer, title, location and date exactly as in the inventory. Never change dates, titles, numbers, team sizes, technologies or outcomes.
   - Reword for the JD's vocabulary only when the inventory already says the same thing (e.g. "ATO process" if the inventory says Authority to Operate). Never add a skill, tool, credential, metric or responsibility the inventory doesn't contain, even if the JD asks for it.
   - Headline: a short all-caps line matching the target role, built from titles and strengths Eric really has.
   - Competencies: at most 12, only ones the inventory evidences, ordered by JD relevance.
   - Career highlights: 3 to 5, the strongest matches to the JD's required items.
   - Experience: all roles stay, newest first. Give the most bullets to the roles that best match the JD, trim older or less relevant roles to 1-3 bullets, and drop a role's summary before dropping a strong bullet.
   - Education and certifications: include as in the inventory (certification credential IDs are optional; leave them out to save space).
5. **Build:** run `build_resume.py` with `--pdf`. If `pages` isn't 2, adjust and rebuild: over 2 pages, trim the least relevant bullets and shorten summaries; under 2, restore relevant bullets from the inventory (never pad with filler). Repeat until it's exactly 2 pages. Look at the PDF (convert to an image) to check nothing is broken: dates line up at the right margin, no sample text, no stray blank page.
6. **Check truthfulness.** For every bullet, headline, highlight and competency in the output, confirm it traces to a line in `Eric-Full-Resume.md`. Fix or remove anything that doesn't. Do not skip this even when running unattended.
7. **Deliver and save.** Name files `Eric_Scott_Resume_<Company>_<Role>.docx` and `.pdf` (underscores, no spaces). Save the JD next to it as `Eric_Scott_Resume_<Company>_<Role>_JD.txt` (pasted text, or the fetched text with its source URL on the first line) so it's there if an interview follows. On Eric's computer, write into the folder he names (default: the batch subfolder he's using, or ask once). From the cloud, send the files with SendUserFile and say where to file them. If a tracker row exists for the job, add the resume file name to its Notes; never touch Status.
8. **Tell Eric the gaps.** End with the required JD items the inventory has no evidence for, so he can decide whether the inventory is missing something true (then he adds it to `Eric-Full-Resume.md`) or it's a real gap.

## content.json shape
```json
{
  "name": "ERIC SCOTT",
  "contact": ["phone", "city, state", "email", "linkedin url"],
  "headline": "ALL CAPS ROLE-ALIGNED HEADLINE",
  "summary": "2-4 sentences, all true to the inventory",
  "competencies": ["up to 12 short phrases"],
  "highlights": ["3-5 one-sentence achievements"],
  "experience": [
    {"company": "...", "location": "...", "dates": "...", "description": "one-line company description",
     "roles": [{"title": "...", "location": "...", "dates": "...", "summary": "optional", "bullets": ["..."]}]}
  ],
  "education": [{"degree": "...", "school": "..."}],
  "certifications": [{"name": "...", "detail": "issuer, year"}],
  "additional": ["optional lines"]
}
```
Take contact details from the inventory's contact line, not from memory.

## Known constraints
- Never invents or inflates anything; gaps are reported, not filled.
- Never edits `Eric-Full-Resume.md`, the template, or any earlier tailored resume.
- Never commits the template, the inventory, a generated resume or a JD to the repo (public portfolio repo).
- Never sends a resume or applies to a job; Eric does that.
- If the template or inventory can't be found, or the output can't be brought to exactly 2 pages, say so rather than delivering something off-format.

## End-of-run summary
- Job and files produced (names, where they're saved), page count.
- What was emphasized and what was trimmed.
- Gaps: required JD items with no support in the inventory.
- Anything you couldn't verify.
