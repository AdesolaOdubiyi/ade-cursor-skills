---
name: review-loop
description: Iterate on AI PR review feedback until all comments are addressed or
  the cycle limit is hit. Use when the user says "loop on review", "address review
  comments", "keep going until reviewer is happy", "iterate on PR feedback", or any
  request to repeatedly fix AI reviewer comments. Works with any AI reviewer (GitHub
  Copilot, Cursor, Sidekick, etc.) — not tied to a specific tool.
allowed-tools: Bash Read Edit Glob Grep
---

# Review Loop

Automated workflow for iterating on AI PR review feedback until zero open comments
or max cycles reached.

## Requirements

- `gh` CLI authenticated on the relevant GitHub host
- An open PR on the current branch with AI reviewer comments

## On Load: Assess Current State

Do NOT ask the user what they want — gather context first:

```bash
git branch --show-current
gh pr view --json number,url,state,isDraft,reviews
```

| State | Action |
|---|---|
| No PR for current branch | Ask the user — this skill requires an open PR |
| PR exists, unresolved threads | Jump to Step 1 |
| PR exists, all threads resolved | Request a new review cycle, then wait |
| PR exists, clean review (0 inline comments) | Run post-loop gates |

---

## The Loop

### Step 1: Read open comments

Fetch all unresolved, non-outdated review threads on the PR. Note: thread count
going to zero is NOT a clean review signal — the reviewer may not have reviewed
the latest push yet. Confirm via a new review with 0 inline comments (Step 6).

### Step 2: Classify and score each comment

#### 2a. Classify

| Comment pattern | Category | Fix approach | Preemptive comment |
|---|---|---|---|
| Catch blocks, swallowed errors, missing logging | **error-handling** | Ensure handler is specific, caller gets actionable feedback. Don't just add logging — check scope. | Explain why the catch is scoped as it is, what the caller sees on failure. |
| Missing tests, edge cases | **test-coverage** | Write behavioral tests. Only add if the gap is meaningful. | Usually none — test name is the documentation. |
| "Simplify", reduce nesting, refactor | **simplification** | Preserve functionality exactly. Prefer clarity over brevity. | If keeping current structure, comment *why* the verbosity is intentional. |
| Type safety, invariants, validation | **type-design** | Check if illegal states are representable. Enforce invariants at construction. | Comment the invariant, or *why* a looser type was chosen. |
| Comment accuracy, outdated docs | **documentation** | Comments explain "why" not "what". Remove if zero long-term value. | N/A — the comment is the fix. |
| Style, naming, conventions | **style** | Only fix if clearly violated. | Usually none. |
| Auth, injection, secrets, exposure | **security** | Never half-fix. Fully address the attack surface. | Document the trust boundary and what sanitization exists upstream. |

#### 2b. Score fix confidence (0–100)

Check the learned-patterns file first — known flip-flops and false positives start below 50.

- **90–100**: Obviously correct (typos, clear bugs). Fix immediately.
- **70–89**: Likely correct. Fix, verify no regression. Add a code comment if it involves a tradeoff.
- **50–69**: Ambiguous. Check for flip-flops. If new, fix but **always** add a code comment.
- **Below 50**: Push back. Add a code comment with technical rationale.

### Step 3: Fix

Rules:
- Separate commit per comment for traceability
- **Check for flip-flops** before agreeing: did the reviewer previously ask for the opposite? If so, push back.
- **Push back via code comments, not just PR replies.** Replies get resolved and lost. Code comments persist and prevent re-proposals.

### Step 4: Add preemptive intent comments

This applies to ALL fixes and pushbacks, not just low-confidence ones. AI reviewers
are stateless — each cycle, a fresh context sees the code without prior thread history.
The only durable way to communicate intent is through the code itself.

**Add a code comment when:**
- The fix involves a tradeoff
- The code does something counterintuitive
- The approach was debated in a prior cycle
- The code relies on an upstream guarantee not obvious from local context
- A fallback or default exists whose purpose isn't self-evident
- The fix touches error handling — always explain what the caller sees on failure

**Do NOT add when:**
- The fix is trivial and self-evident
- A test name already documents the behavior
- The comment would just restate the code

**Style:** Explain why, not what. 1–2 lines. Direct: "X because Y" not "We chose X because we thought Y might..."

Examples:
```python
# .get() + explicit ValueError (vs direct []) — callers outside this module
# may pass bad data; direct [] would raise KeyError with no context
result = data.get("key")

# Explicit branch for each state — avoids silently ignoring new states
# added by the API in the future
if state == "open":
    ...
elif state == "dismissed":
    ...
else:
    raise ValueError(f"Unknown alert state: {state}")
```

### Step 5: Self-check before pushing

- Error-handling fixes: new catch blocks aren't overly broad
- Test additions: tests actually test behavior, not just assert mocks return mocks
- Simplification fixes: run tests to confirm functionality preserved
- Security fixes: fully addresses the concern — no partial mitigations
- Cross-fix coherence: fixes don't contradict each other
- Preemptive comments audit: for any non-trivial fix lacking a code comment, add one now

### Step 6: Push + reply + resolve

```bash
git push origin BRANCH
# Reply to each thread:
# - Fixed: "Fixed in HASH — [description]."
# - Pushed back: "Keeping current approach — added code comment at [file:line]."
```

### Step 7: Report cycle results

```
## Cycle N

### Fixed (X items)
- [file:line] {category}: {description} → commit {hash} [+comment: "{text}"]

### Pushed Back (X items)
- [file:line] {category}: {reason} → code comment added

### Preemptive Comments Added (X items)
- [file:line]: "{comment text}" — prevents {what it heads off}

### Status
- Clean review: yes/no
- Cycles remaining: N
```

### Step 8: Wait for CI + request new review

Wait for CI to go green, then request a new review cycle. If CI fails, fix the
build issue and push before requesting review — commenting on broken code wastes
a cycle.

Confirm a clean review: look for a new review submitted **after your latest push**
with **zero inline comments**. Absence of unreplied threads is not enough.

---

## Stopping Condition

Loop ends when **either**:

1. **Clean review** — reviewer submits a new review after your latest push with zero inline comments
2. **Max cycles reached** — after **5 cycles**, stop and surface remaining comments to the human

---

## Post-Loop Gates

### Security pass

After the stopping condition, run a security pass on the final diff. Fix Critical/High
findings immediately and push (run one more review cycle). Note Medium findings in the
PR description. Skip Low/Informational unless trivial.

### Update learned patterns

Append new findings to `.claude/review-learned-patterns.md` in the project root.
Create it if it doesn't exist:

```markdown
# Learned Patterns

## Known Flip-Flops
## Known False Positives
## Effective Fix Approaches
## Repo-Specific Patterns
## Cycle Count Baselines
```

Record:
- **Flip-flops**: Reviewer contradicted itself — push back automatically next run
- **False positives**: You pushed back and reviewer didn't re-raise — confirmed false positive
- **Effective fixes**: Approach that resolved in one cycle
- **Baselines**: `REPO, N files changed, M cycles`

### Final polish

Review all files modified across ALL cycles for redundant code, inconsistent patterns,
or code comments referencing now-resolved issues. If cleanup needed: one "polish" commit,
push, run one final review cycle.
