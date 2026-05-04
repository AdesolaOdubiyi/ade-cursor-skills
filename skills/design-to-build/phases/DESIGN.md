# Design phase

Governs discovery, grilling, and creation of the three canonical documents before implementation.

## Step 1 — Read the codebase before asking anything

Explore the repository deeply before design questions:

- Read `README.md`, any existing `CONTEXT.md`, architecture docs, ADRs, and config that affects behavior.
- Map relevant files, modules, routes, and data paths.
- Identify what is production code vs stub vs dead code vs experiment.

When a design choice depends on a fast-moving framework or library, use web search to verify current behavior and APIs. Do not rely on training data alone for those checks.

Report findings to the user first. Surface surprises, contradictions, and unknowns before asking design questions.

## Step 2 — Grill session

Follow the `/grill-me` skill: one question at a time, wait for an answer, then continue.

For each question:

- Provide a recommended answer.
- Push back like a senior staff engineer if the user’s answer introduces risk, ambiguity, or scope creep.

Use web search when the answer depends on current library or vendor behavior.

Apply `systems-design` for topology, data flow, scale, consistency, failure modes, and operational constraints.

Apply `architecture-and-api` for API contracts, module boundaries, service seams, and backward compatibility.

**If the feature is user-facing or includes any frontend UI:** apply `ux-product` before locking the design. Cover cognitive load, mental models, friction, loading/empty/error/success states, and progressive disclosure. Ask at most one or two targeted UX questions. Skip `ux-product` for purely backend or infrastructure work with no user-facing surface.

Do not proceed until these are resolved:

- Runtime topology
- Data flow
- API contracts (or equivalent integration seams)
- Migration strategy (if this is a refactor)
- Testing strategy
- Pre-migration housekeeping (what must happen before behavior changes)

## Step 3 — Create three documents

1. **`CONTEXT.md`** — Domain language only, following `../templates/CONTEXT-TEMPLATE.md`.
   - No phase tracking.
   - No implementation steps or code-level plans — terminology and meaning only.

2. **`docs/ARCHITECTURE_PLAN.md`** — Full plan, following `../templates/PLAN-TEMPLATE.md`.
   - Must include: numbered decisions (D1, D2, …) with exact agreed wording, target structure diagram, migration phases with numbered checklists and explicit validation criteria, testing standards, risk register, future work.
   - After Design completes, this file changes **only** by ticking checkboxes — no silent scope edits.

3. **`docs/PROGRESS.md`** — Working state, following `../templates/PROGRESS-TEMPLATE.md`.
   - Current phase, active task checklist (copied from the plan), validation criteria, what comes next, hard constraints.
   - **Shrinks** as phases complete: remove completed items; do not strike through completed work.

## Step 4 — Present and confirm

Present all three documents together. Get explicit user approval to enter Build.

If anything changes during review, update all three documents before starting Build.

Remind the user: the next session should begin by reading `CONTEXT.md` and `docs/PROGRESS.md` (and `docs/ARCHITECTURE_PLAN.md` for checkbox alignment).
