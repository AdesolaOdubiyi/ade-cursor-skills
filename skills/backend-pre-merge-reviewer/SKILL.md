---
name: backend-pre-merge-reviewer
description: |
  Rigorous backend security and quality review before any merge or commit.
  Activate when user says: review this, pre-merge check, backend review,
  check before merging, audit this change, review my backend code.
---

# Backend Pre-Merge Reviewer

## Citation Rule
When citing a specific line in a finding, verify it first with grep or a file read. Never include an unverified line number. Format: `ClassName.java:L<N>` (or equivalent for the language) with the command that confirmed it.

## Context
You are reviewing AI-generated backend code destined for production.
AI-generated code is statistically likely to contain plausible-looking but
subtly broken logic — especially around auth, error handling, and data boundaries.
Treat every line with suspicion. This code is mission-critical. Do not hold back.
A missed finding here is a production incident waiting to happen.

## Review Standard
Only report findings with: code evidence + realistic production risk + business/system impact.
No theoretical findings. No inflated severity. No handwaving.

Before suggesting a pattern change, check whether the existing codebase already uses a different pattern for the same case. If an established pattern exists, either align to it or surface the divergence explicitly. Do not invent a new pattern when the codebase already solved the problem.

---

## Step 1: Fetch Diff

If given a PR URL:
```bash
gh pr diff <PR_NUMBER> -R <OWNER/REPO>
gh pr view <PR_NUMBER> -R <OWNER/REPO> --json title,body,files,additions,deletions,changedFiles
```

If on a local branch:
```bash
git diff $(git merge-base HEAD main)...HEAD
git diff $(git merge-base HEAD main)...HEAD --name-only
```

### Handling Large Diffs
If `gh pr diff` fails or the diff is too large to read in full:
1. Get the file list: `gh pr view --json files` or `git diff ... --name-only`
2. Read the highest-risk files individually (auth, data access, API handlers, config)
3. Note in the report that the review is partial and which files were read

---

## Step 2: Determine Relevant Modules

Scan the diff and file list. Use these signals to decide which modules to run:

**Always-on** (any non-trivial change):
- Correctness
- Error Handling
- Security
- Style

**Conditional — only run if signal is present:**

| Signal in diff | Module |
|---|---|
| Message queue consumers, event handlers, `@KafkaConsumer`, async workers | Async / Messaging |
| `synchronized`, `volatile`, `AtomicReference`, thread pool config, `ExecutorService` | Concurrency |
| DB migration files, schema changes, Flyway/Liquibase, `*.sql` in migration dirs | DB Migrations |
| Public method signatures changed, new API endpoints, interface changes | API Contracts |
| Test files modified or missing for new behavior paths | Test Coverage |
| New external HTTP calls, RPC calls, webhook receivers | Blast Radius |

When workflow signals appear (new user-visible behavior paths, external writes, async workers), the Test Coverage module must evaluate whether acceptance or integration tests should be added — not just unit tests.

---

## Step 3: Dispatch Parallel Analysis

Dispatch one subagent per relevant module. All run in parallel. Each subagent receives:
- The full diff (or targeted file content if large-diff fallback)
- The list of changed files
- The PR description (if available)
- Its module-specific focus (see Module Prompts below)

**Dispatch all in a single message** to maximize parallelism.

### Module Prompts

**Correctness** — Does the code do what it claims? Trace request lifecycle: `input → validation → auth → logic → storage → external calls → errors → response → logs`. Look for: auth gaps, race conditions, contract drift (breaking existing callers), partial failure states, incorrect assumptions about inputs.

**Error Handling** — Are all failure modes handled? Look for: swallowed exceptions, raw error propagation to clients, missing timeouts, retry without backoff, cascading failure vectors.

**Security** — Auth bypass, injection (SQL/command/SSRF), secret/credential handling, data boundary violations (what leaks to the client that shouldn't), privilege assumptions.

**Style** — Naming consistency, unnecessary complexity, dead code, patterns that diverge from the established codebase without justification.

**Async / Messaging** — At-least-once vs exactly-once semantics, idempotency of consumer logic, poison pill handling, DLQ configuration, ordering assumptions.

**Concurrency** — Race conditions between concurrent callers, visibility guarantees, lock ordering, thread-local misuse.

**DB Migrations** — Backwards compatibility (can old code run against the new schema?), migration reversibility, column/index changes on large tables (lock risk), data loss risk.

**API Contracts** — Does this break existing callers? Removed fields, changed types, changed semantics, missing versioning.

**Test Coverage** — Are new behavior paths tested? Are edge cases covered? Should acceptance/integration tests be added for deployed end-to-end verification?

**Blast Radius** — Who depends on what changed? External callers, downstream services, clients that consume this API.

---

## Step 4: Aggregate and Present Report

Collect all module results. Present a unified report:

1. **Summary** — "Reviewed N files across M modules. X MUST FIX, Y SHOULD FIX, Z NITs."
2. **Findings** — grouped by severity tier (MUST FIX first)
3. **Clean modules** — brief list of modules with no issues

### Finding Format
```
[MUST FIX | SHOULD FIX | NIT] [Severity] [Confidence]
path/to/File.java:L42 — description
Evidence: <verified line content>
Impact: <what breaks in production>
```

### Severity
`Critical` → `High` → `Medium` → `Low`. Do not inflate. Critical means production is at real risk right now.

### Verdict Tiers
- `MUST FIX` — blocks merge; shipping as-is causes a production incident or security failure
- `SHOULD FIX` — strong recommendation, not a hard block; technical debt or reliability risk
- `NIT` — style or naming preference; take or leave

**Overall verdict:** `APPROVED` / `APPROVED WITH NOTES` / `BLOCKED` — one sentence on why.

---

## Step 5: Interactive Q&A

After the report, enter follow-up investigation mode. The user may ask about any finding, request additional analysis, or ask to look up callers and related code. Use all available tools to support investigation.

---

## Step 6: AT / Integration Test Trigger

When the diff adds new behavior paths (new endpoints, new async flows, new external integrations), explicitly evaluate:
- Are there existing acceptance or integration tests for this flow?
- Should new ones be added before this merges?
- What scenarios would a real-user or E2E test need to cover?

Surface the answer as a separate "Test Recommendation" block at the end of the report.
