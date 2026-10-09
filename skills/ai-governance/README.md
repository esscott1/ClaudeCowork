# AI governance

Gatekeeper-style skills — guardrail and policy checks that sit in front of or alongside other AI agent activity, rather than doing the work themselves.

| Skill | Trigger | Depends on |
|---|---|---|
| [`sally`](sally/SKILL.md) | On demand: whenever Eric asks to create, change, rename, retire, check in, or install a skill, or says "Sally". Front door: routes status and drift questions to `sally-audit` | Write access to this repo (git on Eric's computer, a cloud session, or his browser); otherwise the JobSearch Project queue |
| [`sally-audit`](sally-audit/SKILL.md) | On demand: "what's installed?", "is X up to date?", "what's drifted?", or when Sally needs the facts before a change | Read access to this repo and visibility of the installed skills; never writes |
