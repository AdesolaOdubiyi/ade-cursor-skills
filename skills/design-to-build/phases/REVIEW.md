# Review Phase

Run this phase after all implementation phases are complete and validated, before shipping
to production. Do not skip any step. Do not merge steps.

---

## Step 1 — Determine review scope

Based on what changed, determine which reviewers apply:

| Change type | Reviewer to run |
|-------------|----------------|
| Any frontend component, page, hook, or UI logic changed | `frontend-pre-merge-reviewer` |
| Any API route, server logic, DB query, or backend module changed | `backend-pre-merge-reviewer` |
| Any change that touches auth, data persistence, user input handling, env vars, or external APIs | `security-auditor` |

When in doubt, run all three.

---

## Step 2 — Frontend pre-merge review

If applicable: read and follow the `frontend-pre-merge-reviewer` skill on all frontend files
modified across all phases of this build.

Do not proceed until verdict is **Safe to merge** or **Safe with fixes**.
If **Do not merge yet**: fix all blocking issues, re-run clean code review on the fixes,
then re-run this step.

---

## Step 3 — Backend pre-merge review

If applicable: read and follow the `backend-pre-merge-reviewer` skill on all backend/API
files modified across all phases.

Same gate: **Safe to merge** or **Safe with fixes** required before proceeding.

---

## Step 4 — Security audit

If applicable: read and follow the `security-auditor` skill.

Run all passes (Phase 0 through final) as defined in that skill.
Do not ship until all `Critical` and `High` findings are resolved or explicitly accepted
by the user with a documented reason.

---

## Step 5 — Final gate

Before shipping:

- [ ] All implementation phase checkboxes ticked in `docs/ARCHITECTURE_PLAN.md`
- [ ] All reviewer verdicts are **Safe to merge** or **Safe with fixes** (fixes applied)
- [ ] All security audit `Critical` and `High` findings resolved
- [ ] `docs/PROGRESS.md` reflects "All phases complete — ready to ship"
- [ ] Production deployment validated with smoke tests

After shipping:
- Archive or delete `docs/PROGRESS.md`
- `docs/ARCHITECTURE_PLAN.md` remains as a permanent record
- `CONTEXT.md` remains as a permanent domain language reference
