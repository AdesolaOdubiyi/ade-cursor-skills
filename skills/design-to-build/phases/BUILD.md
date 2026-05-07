# Build phase

Governs phased implementation after Design is approved. Each architecture-plan phase is completed in a tight loop before ticking checkboxes.

## Before every build session

1. Read `CONTEXT.md`, `KNOWLEDGEBASE.md`, `docs/PROGRESS.md`, and `docs/ARCHITECTURE_PLAN.md`.
2. State explicitly: **I am reading and following the `/tdd` skill for this session.**
3. Read and follow the `/tdd` skill for all implementation work in this session.

## Per-phase build loop

Repeat for **every** phase listed in `docs/ARCHITECTURE_PLAN.md` until all implementation phases are complete.

### Step 1 — Implement (TDD)

- Follow `/tdd` strictly: **one test → one implementation** (vertical slices only).
- Tests verify behavior through **public interfaces** only.
- Cover edge cases required by the plan for this phase.
- Run the test suite **multiple times** — a single passing run is not confirmation.
- No speculative features; if it is not in the plan, it goes under **Future Work** in the architecture plan instead.

### Step 2 — Clean code review

- After all tests pass for this slice/phase work, read and follow **`clean-code-reviewer`** on every file touched in this phase.
- Fix every **Block** violation.
- Fix **Fix** violations unless explicitly documented as waived with user approval.
- Do not proceed with unresolved **Block** violations.

### Step 3 — Validate

- Run the **validation criteria** for the current phase from `docs/ARCHITECTURE_PLAN.md`.
- Run **all** tests multiple times again.
- If validation fails: fix, re-run clean code review on the fixes, re-run validation until stable across multiple runs.
- Do **not** tick the phase checkbox in `docs/ARCHITECTURE_PLAN.md` until validation passes across multiple runs.

### Step 4 — Update documents

- Tick the completed checkboxes in `docs/ARCHITECTURE_PLAN.md` for this phase only.
- Remove completed tasks from `docs/PROGRESS.md` (shrink the file — do not strike through).
- Promote the next phase to active in `docs/PROGRESS.md` (current phase, active checklist, validation, what is next).
- Update `KNOWLEDGEBASE.md` only when a design decision is confirmed, a new hard constraint appears, or product/engineering context changes in a way future sessions must remember.

## Phase 0 — Housekeeping rule

If the current phase is **housekeeping**:

- **Zero** behavior changes.
- Git diff shows only renames, deletes, moves, config-only edits, or comments — **no logic changes**.
- Build/tests must pass clean before the phase is ticked complete.

## Hard rules during build

- Do not re-open locked decisions without an explicit amendment recorded in the plan and user confirmation.
- Do not ship features not listed in the plan — add to **Future Work** instead.
- Do not disable type checking or relax static checks without explicit user approval and a documented reason in `docs/ARCHITECTURE_PLAN.md` or `docs/PROGRESS.md`.
- Always use terminology from `CONTEXT.md` in new code, tests, and docs.
