---
name: "job-fit-rubric"
description: "Shared scoring rubric for Eric's job-search skills, not run on its own: job-lead-intake-scan, job-lead-tier2-scoring and job-lead-manual-jd-lookup load it whenever they set a Fit Score. Defines the evidence table the model fills in and runs score.py to turn it into a deterministic 1-10 score with a readable breakdown."
---

# Job Fit Rubric (v1.0)

The single source of truth for how a Fit Score is produced. Scoring is split in two:

1. **Judgment (the model):** read the job and Eric's master resume, and classify each item into a small evidence table. This part is still a model's reading, but every classification is written down and can be checked.
2. **Arithmetic (`score.py`):** turn that table into a 1-10 score. Same table in, same score out, every time. The weights are constants at the top of `score.py`, and this file explains them.

Never pick a Fit Score by feel, and never adjust the number `score.py` returns. If the score looks wrong, the evidence table is wrong: fix the classification, re-run, and say what you changed.

## Files in this skill
- `score.py`: the scoring function. Run `python3 <this skill's directory>/score.py evidence.json` (or pipe JSON on stdin). It prints `score`, `raw`, `caps_applied`, `breakdown` and `rubric_version`, or exits 2 with the reason if the table is invalid.
- `test_score.py`: checks that the rules below hold (`python3 -m pytest -q` in this directory).

## Step 1: build the evidence table

Write a JSON object:

```json
{
  "mode": "tier2",
  "title_seniority": "strong | partial | weak",
  "work_arrangement": "remote | colorado_onsite | colorado_hybrid | relocation_required | unknown",
  "requirements": [
    {"text": "<quoted from the JD>", "type": "required | preferred", "evidence": "strong | partial | none", "big2_evidence": "<short quote or pointer from Big2, empty if none>"}
  ]
}
```

Use `"mode": "tier1"` and omit `requirements` when no job description has been read (title, company and location from an alert email only).

**`requirements` (Tier 2 only).** One entry per distinct qualification in the JD.
- `text`: the JD's own words, trimmed. Don't paraphrase.
- `type`: `required` for anything under "Requirements", "Qualifications", "Must have", "You have", or stated as required, or as a minimum years figure. `preferred` for "Preferred", "Nice to have", "Bonus", "A plus", "Ideally". If the JD doesn't separate them, treat its main qualifications list as required and anything hedged ("familiarity with", "exposure to") as preferred.
- `evidence`, judged against Big2 only, never against what Eric might know but hasn't written down:
  - `strong`: Big2 directly shows it (same tool, method, certification, or responsibility, and at least the years asked for).
  - `partial`: related or adjacent evidence (a comparable tool, fewer years, done but long ago, or only certification without hands-on delivery).
  - `none`: nothing in Big2 supports it.
- Combine near-duplicates (the same ask in two bullets is one entry). Leave out boilerplate that isn't a qualification (benefits, EEO, "passion for our mission"). A JD normally yields 6-20 entries.

**`title_seniority`.** Compare the role's title and level with Eric's background (enterprise/solutions architecture, IT PMO and delivery leadership, technical program/project management, AI enablement, cloud/infrastructure consulting; senior/lead/principal/manager level).
- `strong`: same kind of role at the same level.
- `partial`: right family but a level off (e.g. a junior or a VP role), or an adjacent role (e.g. business analyst for a PM).
- `weak`: a different discipline or a big level mismatch.

**`work_arrangement`.** Eric lives in Colorado and will not relocate.
- `remote`: fully remote within the US, including occasional travel.
- `colorado_onsite` / `colorado_hybrid`: the office is in Colorado, within a reasonable daily commute for Eric. If unsure whether a Colorado office is commutable, use `unknown` and say so in Notes.
- `relocation_required`: on-site or hybrid anywhere outside Colorado, or "remote" but requiring residence in a specific state other than Colorado.
- `unknown`: the posting or email doesn't say. Prefer finding out over guessing.

## Step 2: run the script

Write the table to a scratch file and run `score.py`. If it exits 2, fix the table and run it again. Copy `score` into the Fit Score column and the `breakdown` string into Notes, exactly as returned.

## The rules `score.py` applies (v1.0)

| Component | Weight | How it's measured |
|---|---|---|
| Requirements coverage | 70% | Each requirement earns 1 (strong), 0.5 (partial) or 0 (none). **Required items count 3×, preferred items 1×.** Coverage = weighted earned ÷ weighted possible. |
| Title / seniority | 20% | strong 1, partial 0.5, weak 0 |
| Work arrangement | 10% | remote, CO on-site and CO hybrid 1; unknown 0.5; relocation 0 |

`raw = 10 × (0.7 × coverage + 0.2 × title + 0.1 × arrangement)`, rounded half-up to a whole number between 1 and 10. Then these ceilings apply:

- **Relocation required → at most 3.** Eric won't move, so no other strength can lift it.
- **Fewer than half of the required items met (counting partials as half) → at most 5.**
- **Tier 1 (no JD read) → at most 8.** Requirements coverage is assumed to be 60%, since nothing has been checked yet.

The resulting colour bands on the tracker are unchanged: 8-10 strong, 6-7 workable with gaps, 1-5 stretch.

**Example breakdown in Notes:** `Rubric v1.0 | Req 1.5/3 · Pref 1/2 · coverage 50% · Title strong · Remote | raw 6.5 → 7`

## Changing the rubric
Weights and caps live only in `score.py`. Change them there and in the table above in the same PR, bump `RUBRIC_VERSION`, and run the tests. Scores already in the tracker keep the version they were scored under (it's in their Notes); rescoring old rows is a separate, explicit request.

## Known constraints
- Scores against Big2 only. If Eric has experience that isn't in Big2, it isn't evidence until he adds it to Big2 himself.
- This skill never touches the tracker; the calling skill does.
- Gap selection (Gap 1-3) is defined by each calling skill, not here.
