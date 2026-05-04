# Review phase

Runs after **all** implementation phases in `docs/ARCHITECTURE_PLAN.md` are complete and ticked, and before shipping to production.

## Step 1 — Determine scope

- **Frontend** changed (UI, client bundles, client routing, styles) → run **`frontend-pre-merge-reviewer`**.
- **Backend / API / DB** changed → run **`backend-pre-merge-reviewer`**.
- **Auth, persistence, user input, environment variables, or external APIs** changed → run **`security-auditor`**.
- When in doubt, run **all three** reviewers.

## Step 2 — Frontend pre-merge review

- Read and follow **`frontend-pre-merge-reviewer`**.
- Proceed only on verdict **Safe to merge** or **Safe with fixes** (with fixes applied and re-reviewed).
- If verdict is **Do not merge yet** (or equivalent blocking state): fix all blocking issues, re-run **`clean-code-reviewer`** on touched files, then re-run this step.

## Step 3 — Backend pre-merge review

- Read and follow **`backend-pre-merge-reviewer`**.
- Same gate as Step 2: **Safe to merge** or **Safe with fixes** only.

## Step 4 — Security audit

- Read and follow **`security-auditor`** and complete all passes defined by that skill.
- Do not ship until all **Critical** and **High** findings are resolved **or** explicitly accepted by the user with a written reason recorded in `docs/PROGRESS.md` or `docs/ARCHITECTURE_PLAN.md` (append-only note).

## Step 5 — Final gate checklist

Confirm all are true before ship:

- [ ] All implementation phase checkboxes are ticked in `docs/ARCHITECTURE_PLAN.md`
- [ ] All reviewer verdicts are **Safe to merge** or **Safe with fixes** (and fixes are merged)
- [ ] All **Critical** and **High** security findings are resolved or explicitly accepted with documented reason
- [ ] `docs/PROGRESS.md` reads **All phases complete — ready to ship** (or equivalent explicit statement)
- [ ] Production deployment validated with smoke tests (or the project’s defined release verification)

## After shipping

- Archive or delete `docs/PROGRESS.md`.
- Keep `docs/ARCHITECTURE_PLAN.md` and `CONTEXT.md` permanently as the historical record of decisions and language.
