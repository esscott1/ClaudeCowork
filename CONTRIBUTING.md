# How this repo is maintained

Changes are drafted through a chat with Claude, then landed here on their own feature branch and opened as a pull request rather than pushed straight to `main` — same review-before-merge discipline as any other codebase, even though the "codebase" is a set of skill instructions. `main` only moves when a PR is merged. Pull with `git pull` to sync your local clone whenever you want the latest merged state.

## Adding a new skill

1. Pick the category it belongs to under `skills/` (`personal-productivity`, `pm-enterprise`, `developer-tools`, `ai-governance`) — or propose a new one if it genuinely doesn't fit any of those.
2. Follow the conventions in [`docs/skill-template.md`](docs/skill-template.md) for the SKILL.md itself.
3. Add or update that category's `README.md` table with the new skill, trigger, and dependencies.
4. If the skill changes what's in the root `README.md`'s category index (a brand-new category, say), update that too.
