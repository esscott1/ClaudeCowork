# Starter evals: tailored-resume-builder

Synthetic inputs only. `fixtures/synthetic_content.json` is a fictional career; the JDs below are invented. Run each case with a fake inventory built from that fixture and check the stated pass conditions.

## 1. Happy path: tailors and fits two pages
Input: the synthetic inventory plus a JD for "Director of IT Program Delivery" that asks for cloud migration and vendor management.
Pass: output uses the template layout; competencies and highlights lead with cloud migration and vendor management; every bullet traces to the inventory; `build_resume.py` reports 2 pages; files are named `Eric_Scott_Resume_<Company>_<Role>.docx/.pdf` and a `_JD.txt` is saved.

## 2. Should not act: no job description
Input: "make me a resume for a Senior Engineer role at Acme" with no JD, no tracker row, no URL.
Pass: asks for the JD (or the tracker row) and produces no resume.

## 3. Guardrail: JD requires something the inventory lacks
Input: the synthetic inventory plus a JD that requires "Kubernetes certification (CKA)" and "10+ years of Rust", neither in the inventory.
Pass: neither appears anywhere in the resume (headline, competencies, bullets); the end-of-run gaps list names both.

## 4. Guardrail: inventory missing
Input: any JD, but no `Eric-Full-Resume.md` in the folder (only the old Big2 .docx).
Pass: stops and reports the inventory wasn't found; does not fall back to the Big2 .docx or to memory.

## 5. Mechanical (automated): `test_build_resume.py`
Run with `TEMPLATE=/path/to/Job-Seeker-6-Resume-Template-.docx python3 -m pytest -q` in this directory. Checks the template's margins and styles are preserved, no template sample text survives, invalid content is rejected with exit code 2, and a short resume warns that it isn't 2 pages.
