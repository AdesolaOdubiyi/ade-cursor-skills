---
name: build-with-confidence
description: End-to-end feature implementation with test-driven development, multi-pass review loops, acceptance verification, and a two-pass context consolidation (update docs, then readonly completeness subagent audit until clean). Switches to Opus 4.6, researches before building, runs TDD vertical slices, loops pre-merge reviews until all pass, then validates with acceptance tests. Use when implementing new features, building modules, or tackling complex requirements that demand confidence.
---

# Build with Confidence

A comprehensive, orchestrated workflow for implementing features safely. Combines context confirmation, research, TDD vertical slices, automated review loops, and acceptance testing.

## Overview

This skill runs a 6-phase build cycle (Phase 6 has two passes — consolidate, then audit):

1. **Context & Research** — Confirm what you're building, web-search relevant patterns
2. **TDD Vertical Slices** — Write failing tests, implement, verify green (happy path → edge cases)
3. **Review Loop** — Run backend pre-merge + security auditor until all pass
4. **Acceptance Testing** — Run end-to-end tests, validate user workflows
5. **Ship** — Confident that code is correct, tested, secure, and works
6. **Context Consolidation** — Pass 1: update docs intelligently; Pass 2: completeness subagent audit + reconciliation loop until clean

**Model**: Opus 4.6 (upgraded automatically at the start).

---

## Quick Start

```
/build-with-confidence <feature-description>
```

Example:
```
/build-with-confidence Implement user password reset flow with email verification
```

---

## Phase 1: Context & Research

### Confirm What You're Building

Answer these questions:

- **What is the feature?** (user-facing or internal?)
- **Why does it exist?** (business need, user pain point, tech debt?)
- **What's the scope?** (files touched, dependencies, integrations?)
- **Success criteria?** (acceptance tests, performance targets, backwards compat?)

Claude will ask clarifying questions and document the shared model.

### Web Research Phase

Before any code, Claude searches for:
- Current best practices (framework patterns, libraries)
- Common failure modes (security, perf, edge cases)
- Reference implementations (if applicable)

Research feeds into the test strategy and implementation plan.

---

## Phase 2: TDD Vertical Slices

Write one test at a time, implement it, verify green. Each slice is a mini-feature.

### Slice Checklist

For each vertical slice, you test:

- **Happy path** — Main success flow (user does the thing, it works)
- **Empty state** — No data / default behavior (empty list, null input, first run)
- **Partial state** — Degraded conditions (some data missing, retry, fallback)
- **Timeout handling** — Async operations that stall or fail
- **Invalid input** — Bad user data, type mismatches, constraint violations
- **Domain edges** — Boundary conditions specific to the feature

### Workflow

1. **Write failing test** for one scenario (e.g., happy path)
2. **Implement** the minimal code to make it pass
3. **Verify green** (test passes, no regressions)
4. **Repeat** for the next scenario (empty state, invalid input, etc.)
5. **Refactor** once all tests for a slice are green

Typical slices per feature:

- Small feature: 3–4 slices (happy path, two edge cases, invalid input)
- Medium feature: 5–7 slices (add empty state, timeout handling, partial state)
- Large feature: 8+ slices (full matrix of conditions)

---

## Phase 3: Review Loop

Once all TDD slices are green, run two sub-agents in a loop:

1. **Backend Pre-Merge Reviewer** — Security, errors, contracts, efficiency
2. **Security Auditor** — OWASP, secrets, data flow, permissions

### Loop Until All Pass

- Review finds issues → Claude fixes them
- Re-run reviews on the fix
- Repeat until both agents have **zero findings**

This uses parallel agents and converges on a clean state.

---

## Phase 4: Acceptance Testing

Run end-to-end tests against the real feature:

- **User workflows** — Complete scenarios from user perspective
- **API contracts** — Requests/responses match spec
- **Integration points** — Dependencies work correctly
- **Smoke tests** — Healthy defaults, no crashes

If acceptance tests fail, loop back to Phase 2 (TDD) to fix the root cause.

---

## Phase 5: Ship

Once acceptance tests pass and reviews are clean:

- Create a Git commit (if not already made during implementation)
- Prepare a PR summary
- Mark the feature as **Ready**

---

## Phase 6: Context Consolidation (every build)

**Trigger:** End of every build cycle, before handing off to the next session.

Context is **unique per project**. Do not treat all repos the same — pick a strategy that fits the project's doc layout and memory needs (see [examples/context-strategies/](examples/context-strategies/)).

### Principles (from memory-systems research)

- **Consolidate, don't append.** Replace stale sections; merge duplicates; resolve contradictions. Avoid unbounded growth in `CONTEXT.md`, `STATUS.md`, ADRs.
- **Keep episodic + semantic layers.** Raw build artifacts (JSONL logs, smoke outputs, handoff notes) are episodic; distilled facts go into semantic docs (`CONTEXT.md`, `DECISIONS.md`, `STATUS.md`).
- **Conflict resolution:** When new facts contradict old docs, **recency wins** for operational status; **ADRs win** for intentional decisions unless explicitly superseded.
- **Prune aggressively:** Remove "planned" items that shipped; archive one-off research; delete duplicate bullet lists across files.

### Per-build checklist

1. **Read** current `CONTEXT.md` (or equivalent), `STATUS.md`, and any project-specific memory files.
2. **Extract** what changed this build: features shipped, blockers discovered, decisions made, tests/smokes run.
3. **Update semantic docs:**
   - `STATUS.md` — checkmarks only; no narrative duplication
   - `CONTEXT.md` — product scope, invariants, domain terms (stable truths)
   - `DECISIONS.md` / ADRs — only when a decision should not be re-litigated
   - `RUNBOOK.md` — operator commands that changed
4. **Append episodic record** (optional but recommended): one line or short entry in `docs/build-log/YYYY-MM-DD.md` or project-specific log (e.g. `batch_results.jsonl` for runtime outcomes).
5. **Handoff** (multi-session work): write or refresh `docs/HANDOFF.md` with goals, current state, next tasks, and commands — so a fresh agent needs minimal chat history.

### Project-specific context strategies

| Project type | Primary context | Episodic | Strategy |
|--------------|-----------------|----------|----------|
| Product app (e.g. AIHawk ADE) | `CONTEXT.md` + `docs/STATUS.md` | `docs/build-log/`, `batch_results.jsonl` | Consolidate STATUS after each gate; keep CONTEXT stable |
| Greenfield feature | `docs/ARCHITECTURE_PLAN.md` + `PROGRESS.md` | Session handoffs | Merge PROGRESS into STATUS when feature ships |
| Library / SDK | README + API docs | CHANGELOG | Update CHANGELOG; trim README duplication |
| Infra / CI | Runbooks + ADRs | CI logs, incident notes | ADR for irreversible choices; runbook for commands |

See worked examples in [examples/context-strategies/](examples/context-strategies/).

### What NOT to do

- Do not add a new section for every build without editing old sections
- Do not copy entire chat transcripts into context files
- Do not duplicate the same fact in CONTEXT, STATUS, and HANDOFF — link or pick one canonical home

### Pass 2: Completeness audit (subagent + reconciliation loop)

**Do not skip.** After Pass 1 edits are done, launch a **specialized read-only subagent** whose **only job** is to verify nothing was missed in the build or context docs. The main agent then reconciles every finding until the subagent reports a clean state.

This mirrors Phase 3 (review loop) but targets **build completeness** and **context accuracy**, not code security.

#### Launch the subagent

Use the Task tool (or equivalent subagent spawn) with:

- **subagent_type:** `explore` (preferred — read-only, codebase + docs walk) or `generalPurpose` with `readonly: true`
- **Single responsibility:** Completeness audit only — no implementation, no doc edits
- **Prompt:** Copy from [examples/completeness-audit-prompt.md](examples/completeness-audit-prompt.md), filling in:
  - Feature / build scope for this session
  - Files the main agent claims to have changed
  - Project context strategy (from `examples/context-strategies/` if applicable)

#### What the subagent must verify

| Area | Checks |
|------|--------|
| **Build vs docs** | Every shipped feature in code has a matching `[x]` or accurate `[~]` in STATUS; no "not built yet" where code exists |
| **Docs vs build** | No STATUS/HANDOFF claims for features that don't exist in the repo |
| **Cross-file consistency** | Same fact not contradicted across CONTEXT, STATUS, HANDOFF, RUNBOOK, DECISIONS |
| **Stale content** | "Planned", "next", "TODO" language removed where work shipped; old percentages/progress replaced |
| **Operator gap** | RUNBOOK commands match actual CLI/module paths; env vars documented if new |
| **Episodic layer** | build-log entry exists for this session if the project uses one |
| **Handoff readiness** | HANDOFF.md (if used) lists correct next tasks, commands, and blockers — not session chatter |
| **Test evidence** | Tests mentioned in docs were actually run; failing tests not marked shipped |
| **Scope bleed** | Unrelated files changed without mention; requested features left incomplete |

#### Subagent output format (required)

The subagent returns **findings only**, structured as:

```
COMPLETENESS AUDIT — <pass N>

CLEAN: no | yes

BUILD GAPS (code shipped, docs missing/wrong):
- [severity: high|medium|low] <finding> → evidence: <file:line or path>

CONTEXT GAPS (docs wrong/stale/contradictory):
- [severity: high|medium|low] <finding> → evidence: <file:line or path>

MISSED SCOPE (requested but not done):
- [severity: high|medium|low] <finding> → evidence: <what was asked vs what exists>

DUPLICATION / NOISE:
- [severity: low] <finding> → evidence: <paths>
```

If truly nothing missed: `CLEAN: yes` with empty finding sections.

#### Main agent reconciliation loop

1. Read subagent findings
2. **Fix or dismiss with reason** — each high/medium finding gets a doc or code change; low findings fixed or explicitly accepted
3. **Re-launch the same subagent** with updated file list and a note of what was reconciled
4. **Repeat until** subagent returns `CLEAN: yes`
5. **Cap at 3 audit rounds** — if still not clean, summarize remaining gaps for the user instead of looping forever

The main agent owns reconciliation; the subagent never edits files.

#### When Pass 2 fails open

If subagent tooling is unavailable, the main agent performs a **manual completeness checklist** using [examples/context-strategies/consolidation-checklist.md](examples/context-strategies/consolidation-checklist.md) plus a self-review of git diff vs STATUS/HANDOFF.

---

## Workflow Phases (Internal)

Claude orchestrates this using sub-agents:

| Phase | Agents | Success Criteria |
|-------|--------|------------------|
| Context | Griller, WebSearcher | Shared model documented, research findings logged |
| TDD Slices | TDD Agent (runs inline) | All slices green, no regressions |
| Review Loop | Backend Reviewer + Security Agent | Zero findings from both |
| Acceptance | Test Agent | All workflows pass, no manual failures |
| Ship | Git Agent | Commit created, PR ready |
| Context Consolidation Pass 1 | Inline | Semantic docs updated, stale content replaced |
| Context Consolidation Pass 2 | Completeness Audit subagent (`explore`, readonly) | `CLEAN: yes` after main-agent reconciliation loop |

---

## Advanced: Custom Slices

If the default checklist doesn't fit your domain:

- Add domain-specific slices (e.g., **payment processing**: PCI compliance, refund flows, duplicate detection)
- Mention them in the initial context step — Claude will adapt the test matrix

---

## When to Use

✅ Implementing new features  
✅ Building internal modules or services  
✅ Refactoring with high confidence needed  
✅ Security-sensitive code (auth, payments, data handling)  
✅ Complex business logic with many edge cases  

❌ One-off scripts or throwaway code  
❌ Trivial bug fixes (use `/code-review` instead)  
❌ Documentation changes (use `/technical-docs-writer`)  

---

## Example: Password Reset Feature

```
Phase 1: Context & Research
  ✓ Feature: Secure password reset with email verification
  ✓ Scope: Auth service, email provider, user DB
  ✓ Research: NIST password guidelines, rate limiting patterns

Phase 2: TDD Slices
  ✓ Slice 1: Happy path (request reset → send email → verify token → new password)
  ✓ Slice 2: Invalid email (nonexistent user)
  ✓ Slice 3: Expired token (token older than 24 hours)
  ✓ Slice 4: Rate limiting (too many reset requests)
  ✓ Slice 5: Concurrent resets (user resets twice before clicking first email)

Phase 3: Review Loop
  ✓ Backend review: No SQL injection, error messages don't leak user existence
  ✓ Security review: Token entropy, rate limiting, session handling
  ✓ Loop until clean

Phase 4: Acceptance Testing
  ✓ Happy path end-to-end (browser → email → reset)
  ✓ Invalid workflows (expired link, tampered token)

Phase 5: Ship
  ✓ Commit: "Add secure password reset with email verification"
  ✓ PR: Links to acceptance tests, security review summary

Phase 6: Context Consolidation
  Pass 1 (main agent):
    ✓ STATUS.md: mark password reset shipped; remove "planned" bullets
    ✓ CONTEXT.md: add invariant "reset tokens expire in 24h" (replace old TTL note)
    ✓ build-log entry: smoke URLs + test counts
    ✓ HANDOFF.md refreshed if work continues next session
  Pass 2 (completeness subagent → reconcile loop):
    ✓ Subagent audit: RUNBOOK has reset CLI; STATUS matches routes; no stale "planned"
    ✓ Main agent fixed 2 medium findings (HANDOFF command typo, missing build-log)
    ✓ Re-audit → CLEAN: yes
```

---

## FAQ

**How long does this take?**  
Depends on feature size. Small features: 30–45 minutes. Medium: 1–2 hours. Large: 2–4+ hours. The framework scales.

**Can I skip a phase?**  
Not recommended. Context prevents you from building the wrong thing; research prevents security holes; TDD prevents regressions; reviews catch what tests miss; acceptance testing validates the whole system.

**What if tests fail during the review loop?**  
Loop back to Phase 2 (TDD) and fix the underlying bug. A failing review finding usually means a test was missing.

**Can I skip Phase 6 Pass 2?**  
No. Pass 1 is where the main agent updates docs; Pass 2 is the independent check that catches stale STATUS rows, missing RUNBOOK commands, and scope the main agent forgot. Same rationale as Phase 3 review loop.

**What if the completeness subagent keeps finding issues?**  
Main agent reconciles and re-runs (max 3 rounds). Escalate remaining gaps to the user with evidence — do not mark the build "ready" with open high-severity findings.

**Can I use this for refactors?**  
Yes. Treat the refactored code as the feature. TDD slices ensure the refactor doesn't break behavior.

---

## See Also

- `/tdd` — For deep TDD guidance on a single module
- `/backend-pre-merge-reviewer` — For a comprehensive backend review
- `/security-auditor` — For security-specific audits
- `/clean-code-reviewer` — For code cleanliness after implementation
- `/design-to-build` — For architectural planning before this workflow
- [examples/context-strategies/](examples/context-strategies/) — Per-project context consolidation patterns
- [examples/completeness-audit-prompt.md](examples/completeness-audit-prompt.md) — Subagent prompt for Pass 2 audit
