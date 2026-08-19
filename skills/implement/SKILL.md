---
name: implement
description: |
  Structured implementation loop for any non-trivial feature, tool, or change. Use when you have
  a spec, ticket, or clear requirement and need disciplined execution: confirm context, search
  best practices with cited sources, lock architecture before coding, TDD vertical slices, review
  loop, ship. Use instead of coding directly whenever the feature touches more than one layer or
  requires any design decision.
---

# Implement

## Citation Rule (applies throughout all steps)

When citing any code location, always verify before including it:
- Run `grep -n "<pattern>" <file>` or Read to confirm the exact line number
- Never cite line numbers from search results — they are approximate
- Format: `FileName.ext:L<N>` with the command that confirmed it
- For APIs/docs: cite the URL or the exact source used

---

## Step 1 — Confirm Context

Check for `FEATURE_CONTEXT.md` in the repo root.

**If it exists:** Read it. Verify every key claim against live source before trusting — check that file paths, method signatures, and return types match the actual code.

**If it does not exist:** Synthesize context from available artifacts (README, tickets, prior conversation). Then create `FEATURE_CONTEXT.md` at the repo root with:
- What is being built (one sentence)
- Request/input shape: each field — name, type, where it comes from
- APIs called: exact method signature, return type, file location
- Response/output shape: each field — name, type, why the consumer needs it
- Scope boundary: what this does NOT do

Before presenting the summary, verify and cite:
1. Each client method: exact signature, return type, location (grep-confirmed)
2. At least one sibling model in the same module (to confirm field naming/nullability conventions)
3. The downstream consumer of this output and what they need to act on it

**Checkpoint:** State `"Step 1 done: [what the feature does and what it will NOT call]"`
⏸ STOP. Do not proceed to Step 2 until the user explicitly confirms the contract is correct.

---

## Step 2 — Search Before Building

Dispatch all of the following in parallel:

**A — Codebase patterns:** Find existing callers of the APIs from Step 1. Note usage patterns, error handling conventions, and how similar features handle edge cases in this repo.

**B — Web search (best practices + dissent):** For each major technology/framework/pattern involved, dispatch two sub-searches in parallel:

*Primary — best practices:*
- Query: `"[framework/library] [feature type] [pattern] best practices [current year]"`
- Secondary: `"[technology] [feature] production considerations -tutorial -beginner"`
- For each source, return:
  - URL + title + published date
  - Direct quote (1–3 sentences verbatim, in quotes)
  - Reliability rationale: one sentence on why this source is authoritative (official docs, known maintainer, engineering org blog, etc.)
  - Application: one sentence — how this affects the architecture decision
- Reject: undated tutorials, "top 10" listicles, anything without an identifiable author or org
- Aim for 2–4 cited sources per major decision

*Dissenting — counter-perspectives:*
- Query: `"[technology/pattern] drawbacks alternatives [current year]"` or `"why not [pattern]"`
- Look for genuine engineering tradeoffs, known failure modes, or teams that moved away from this approach and why
- Return the same format: source, quote, rationale, application
- If no credible dissent exists, say so explicitly — don't fabricate balance

**Reconcile**: if primary and dissenting sources genuinely conflict, surface the tension as a decision point in the Architecture Lock step. Engineering is about tradeoffs — don't collapse disagreement prematurely.

**C — Architecture docs/ADRs:** Check `docs/adr/`, `docs/decisions/`, or any architecture docs. Note any decision that constrains this feature. If a planned approach contradicts an existing ADR, surface it as a `⚠️ ADR conflict` before proceeding.

Present findings as a structured list (not a paragraph):

```
• [Technology/pattern]: [key finding] — [source]
• [Dissent]: [counter-perspective, if any] — [source]
• [ADR conflict]: [which ADR, what conflicts] — if applicable
• [Discrepancy from Step 1]: [what differs from the assumed approach] — if applicable
```

**Checkpoint:** State `"Step 2 done: [key pattern or tradeoff surfaced]"`
⏸ STOP. Do not proceed until the user confirms.

---

## Architecture Lock

Compile Steps 1–2 into a complete implementation spec. For each item: the decision, its source (grep-verified or cited), and a one-line why. Update `FEATURE_CONTEXT.md` with this spec.

**Specify:**
- Input model — each field: name, type, source, why it's needed
- APIs called — each: exact signature, return type, file location, why this and not an alternative
- Logic — numbered steps covering the full input-to-output path: calls, transforms, error handling, empty state
- Output model — each field: name, type, description, why the consumer needs this value
- Empty case — what returns when no data exists
- Error handling — each exception type and how it's handled
- Tradeoffs accepted — for any decision where dissent existed in Step 2, record what was chosen and why

**Checkpoint:** State `"Architecture locked: [one sentence — what's being built and what it calls]"`
⏸ STOP. Do not write any code until the user confirms this architecture.

---

## Step 3 — TDD Vertical Slices

Follow the `tdd` skill for all implementation work.

Build in vertical slices — one complete path through the system, fully tested before the next.

**Slice order** (skip slices that don't apply to the domain):
1. Happy path — typical input, populated data
2. Empty state — valid input, no data → graceful response, never throw
3. Partial state — some data present, some absent → partial results vs. fail-all decision
4. Unavailability — downstream timeout or 5xx → never propagate raw errors
5. Invalid input — missing required fields, wrong types → fast rejection with a clear message
6. Negative input — explicitly bad values (negative IDs, zero where positive required, etc.)
7. Overloading input — boundary conditions (max values, very large inputs, empty collections at scale) where relevant
8. Domain-specific edge cases

For each slice:
- Write the failing test first
- Implement the minimum code to make it pass — following the constraints below
- Do not move to the next slice until the current one is green
- Tests must assert on specific fields, not just "response is not null"

**Implementation quality constraints (apply to every slice):**
- Follow existing codebase patterns unless you have a specific reasoned deviation — surface deviations explicitly
- No magic numbers: extract every literal to a named constant at the top of the file
- Functions should do one thing; extract helpers for repeated logic — prefer DRY over copy-paste
- Keep functions small and readable; if a function requires scrolling to understand, split it
- Structure top-to-bottom: public API / orchestration first, private helpers below (Python and most languages)
- No explanatory comments for what the code does; a comment is only justified when the WHY is non-obvious

After all slices are green, run the full test suite.

---

## Step 4 — Review Loop

**Phase 1 (alone first):**
Run `backend-pre-merge-reviewer` (or `frontend-pre-merge-reviewer` for UI changes). Fix every MUST FIX. Re-run until Phase 1 is clean.

**Checkpoint:** State `"Phase 1 clean"`

**Phase 2 (parallel, only after Phase 1 is clean):**
Run `security-auditor` in parallel with any other applicable reviewers. Fix all findings. Re-run until all return clean in the same pass.

**Exit condition:** State `"CLEAN"` only when all reviewers pass in the same round.

⏸ STOP. Present the final diff to the user for review. Do not proceed to Step 5 until the user explicitly approves.

---

## Step 5 — Ship

**PR split decision** — split into stacked PRs if any of:
- Change touches more than one logical layer (models, implementation, tests are separate layers)
- Change spans more than one repo
- A reviewer would need full context to review any single file

**Stacked PR rules:**
- PR 1 branches from main
- PR N branches from PR N-1 (never from main directly)
- Each PR title: `"[N/total] Brief description"`
- Each PR description explains what the next PR depends on

**All PRs opened as drafts.** No exceptions — a non-draft PR notifies reviewers immediately before you've caught your own issues.

If the feature adds a new public surface or changes a documented interface, run `readme-writer` or `technical-docs-writer` before marking any PR ready for review.
