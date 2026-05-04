---
name: design-to-build
description: |
  End-to-end workflow for designing, architecting, and safely building any non-trivial feature,
  refactor, or system. Runs a structured grill session to surface constraints and lock in decisions,
  produces CONTEXT.md (domain language) and docs/ARCHITECTURE_PLAN.md + docs/PROGRESS.md, then
  guides phased TDD implementation with clean code review, pre-merge review, and security audit
  at each gate. Use when starting a large feature, significant refactor, or anything that requires
  design decisions before a single line of code is written.
---

# Design to Build

## Step 1 — Mode Detection

Before anything else, check the repo for existing docs.

**Fresh start** — neither `CONTEXT.md` nor `docs/ARCHITECTURE_PLAN.md` exists:
→ Go to [phases/DESIGN.md](phases/DESIGN.md)

**Resume** — both files exist:
→ Read `CONTEXT.md`, `docs/ARCHITECTURE_PLAN.md`, and `docs/PROGRESS.md`
→ Identify the current active phase in `docs/PROGRESS.md`
→ Jump directly to [phases/BUILD.md](phases/BUILD.md) at the correct phase

---

## Phases

| Phase | Purpose | Sub-skill references |
|-------|---------|----------------------|
| **Design** | Grill session → lock decisions → create docs | `/grill-me`, `/systems-design`, `/architecture-and-api` |
| **Build** | Per-phase TDD implementation + clean code review | `/tdd`, `clean-code-reviewer` |
| **Review** | Pre-merge review + security audit before shipping | `frontend-pre-merge-reviewer`, `backend-pre-merge-reviewer`, `security-auditor` |

Full instructions for each phase:
- [Design session → document creation](phases/DESIGN.md)
- [Per-phase build workflow](phases/BUILD.md)
- [Review and ship workflow](phases/REVIEW.md)

---

## Document Roles

| File | Purpose | Lifecycle |
|------|---------|-----------|
| `CONTEXT.md` | Domain language glossary — canonical terms for this project | Established in Design phase; updated only when new terms are resolved |
| `docs/ARCHITECTURE_PLAN.md` | Full plan: all decisions, phases with checkboxes, risks, future work | Created in Design phase; checkboxes tick as phases complete; nothing else changes |
| `docs/PROGRESS.md` | Active working state: current phase, open tasks, what is next | Created in Design phase; shrinks as phases complete; deleted or archived when done |

Templates for all three documents: [templates/](templates/)
