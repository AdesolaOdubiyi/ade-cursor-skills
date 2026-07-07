# Build with Confidence — Workflow Reference

## Workflow Structure

When you invoke `/build-with-confidence`, Claude executes a multi-phase workflow using sub-agents and parallel processing.

### Phase 1: Context & Research

```
1a. Context Confirmation (sync)
    - User provides feature description
    - Claude asks clarifying questions
    - Documented output: CONTEXT.md (shared model)

1b. Web Research (parallel, async)
    - Search for relevant patterns (framework, language, domain)
    - Fetch reference implementations
    - Compile findings into RESEARCH.md
```

**Success signal**: CONTEXT.md exists and captures scope, success criteria, and stakeholders.

---

### Phase 2: TDD Vertical Slices

```
2a. Test Plan (1 agent)
    - Review context and research
    - Design test matrix (happy path → edge cases)
    - List all scenarios to cover
    
2b. Slice Loop (iterative, inline)
    for each scenario:
      - Write failing test (pytest, Jest, etc.)
      - Implement code to pass
      - Verify green (no regressions)
      - Refactor if time permits
```

**Success signal**: All test files green, coverage report included.

---

### Phase 3: Review Loop

```
3a. Backend Pre-Merge Review
    - Agent reviews code for:
      * Security vulnerabilities
      * Error handling
      * API contracts
      * Performance concerns
    - Returns findings list

3b. Security Auditor Review
    - Agent reviews code for:
      * OWASP top 10 risks
      * Data flow and permissions
      * Secrets management
      * Cryptography usage
    - Returns findings list

3c. Fix Loop
    while findings.count > 0:
      - Claude fixes each finding
      - Re-run both reviewers
      - Findings = new_findings
      
    Exit when findings.count == 0 for both agents
```

**Success signal**: Both agents return zero findings.

---

### Phase 4: Acceptance Testing

```
4a. User Workflows (1–3 agents)
    - End-to-end scenario testing
    - Browser/API client workflows
    - Happy path verification
    
4b. Integration Tests (1 agent)
    - API contracts
    - Database state changes
    - External service mocks
    
4c. Smoke Tests (1 agent)
    - No crashes
    - Healthy defaults
    - Error boundaries respected
```

**Success signal**: All acceptance tests pass.

---

### Phase 5: Ship

```
5a. Commit
    - Stage all changed files
    - Generate commit message from CONTEXT.md + test summary
    - Create commit
    
5b. Summary
    - Generate PR description
    - Link to acceptance tests and review logs
    - Mark feature as Ready
```

**Success signal**: Commit created, PR summary ready.

---

## Parallel Execution Strategy

Sub-agents run concurrently where possible:

```
Phase 1:
  - Context confirmation (blocks research start)
  - Web research (parallel, non-blocking to TDD)
  
Phase 2:
  - Entirely sequential (each test depends on prior green state)
  
Phase 3:
  - Backend + Security reviews run in parallel
  - Fixes run sequentially
  - After each fix, both reviews run again in parallel
  
Phase 4:
  - User workflows + Integration + Smoke tests run in parallel
  - All must pass before Phase 5
  
Phase 5:
  - Commit + Summary sequential
```

---

## Model Upgrade

At the start of the workflow:

```
Current model: Haiku 4.5 (or user's default)
↓
Upgrade to: Opus 4.6 (or Claude Fable if available)
↓
Reasoning: Multi-agent orchestration, code generation, security review
           benefit from larger context window and stronger reasoning
```

The model is restored to the user's default after Phase 5 (or if the user cancels).

---

## Token Budget

This workflow is a **long-running task** (~30 min to 2 hours wall-clock time).

Token estimate by phase:

| Phase | Agents | Tokens (typical) |
|-------|--------|------------------|
| Context & Research | 2 | 15–30k |
| TDD Slices | 1 (inline) | 50–150k |
| Review Loop | 2 (looped) | 40–100k |
| Acceptance | 3 | 20–40k |
| Ship | 1 | 5–10k |
| **Total** | — | **130–330k** |

If the user has set a token budget via `+500k`, the workflow scales to use the full budget (deeper test coverage, more research, more review iterations).

---

## Failure Modes & Recovery

### If Phase 1 (Context) is unclear
- Skill asks follow-up questions
- Loop until CONTEXT.md is solid
- No code written yet — easy to restart

### If Phase 2 (TDD) tests fail
- Claude debugs the test
- Fixes implementation or test
- Re-runs to verify green
- Moves to next slice

### If Phase 3 (Review) findings accumulate
- Claude fixes highest-priority finding first
- Re-runs both reviewers
- If more findings appear, repeat
- Bounded by a max-iterations cap (e.g., 5 loops)

### If Phase 4 (Acceptance) fails
- Skill logs failure + error
- Asks user: debug now, or loop back to Phase 2?
- If loop back: TDD slice added for the failure case

### If Phase 5 (Ship) fails
- Git error or PR creation error
- Skill logs the error and pauses
- User can manually create PR or restart

---

## Configuration

### Per-Project Overrides

Create a `.build-with-confidence.yml` in the project root to customize:

```yaml
# Model upgrade (default: opus-4-8)
model: claude-fable-5

# Review loop max iterations (default: 5)
review_loop_max_iterations: 3

# Acceptance test runner (default: auto-detect)
acceptance_test_command: "pytest integration/ -v"

# Scenarios to always include (default: standard matrix)
scenarios:
  - happy_path
  - empty_state
  - invalid_input
  - timeout_handling
  - domain_edge_case_1: "custom scenario"
  - domain_edge_case_2: "another scenario"
```

---

## Sub-Agent Details

### Context Confirmation Agent
- Type: `general-purpose`
- Asks 3–5 clarifying questions
- Documents scope, success criteria, stakeholders
- Output: `CONTEXT.md`

### Web Research Agent
- Type: `general-purpose` with WebSearch
- Searches for framework patterns, libraries, security guidelines
- Fetches 2–3 reference implementations
- Output: `RESEARCH.md`

### TDD Agent
- Runs inline (not a separate agent)
- Claude writes tests and code in the main conversation
- Output: Test files + implementation files

### Backend Pre-Merge Reviewer
- Type: `backend-pre-merge-reviewer` skill
- Inherits your existing skill
- Output: Findings list (file + line number + severity)

### Security Auditor
- Type: `security-auditor` skill
- Inherits your existing skill
- Output: Findings list (issue + CWE + severity)

### Acceptance Test Agent
- Type: `general-purpose`
- Writes and runs E2E tests
- Output: Test results + pass/fail summary

---

## Monitoring Progress

While the workflow runs, check `/workflows` to see:

- Current phase (Context, TDD, Review, Acceptance, Ship)
- Active agents and their progress
- Token usage and time elapsed
- Any pauses or user prompts

---

## Exiting Early

If you need to stop the workflow:

1. **During Phase 1–2** (safe): No code committed yet. Restart anytime.
2. **During Phase 3** (safe): Code exists locally; revert with `git reset --hard` if needed.
3. **During Phase 4–5** (destructive): Code may be committed. Use `git log` to review before proceeding.

To pause and resume later:
- Use `/schedule` to resume after a set time
- Or manually restart Phase 2 with the TDD agent

---

## Customization for Your Team

If your team has specific review criteria:

1. Document them in `CONTEXT.md` (phase 1)
2. Mention them in the feature description
3. Backend + Security reviewers will inherit your standards

Example:
```
Context excerpt:
  Team standards:
  - Must use prepared statements for all DB queries
  - Error messages must not leak internal state
  - All external API calls must have timeouts
```

Both reviewers will check these explicitly.
