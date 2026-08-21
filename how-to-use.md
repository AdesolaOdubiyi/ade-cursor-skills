# AI Skills & Rules — How to Use

A personal toolkit of rules and agent skills built for production-quality
engineering work. Covers the full stack with a backend-heavy bias.

Works with Cursor, Claude Code, Codex, Windsurf, and any agent that supports
SKILL.md-style skill files or equivalent rule injection.

## Credits

Several skills in this setup are authored by [Matt Pocock](https://github.com/mattpocock/skills)
and used here with attribution. These include `improve-codebase-architecture`,
`grill-with-docs`, `write-a-skill`, and `caveman`. His full skills library and further information he provides about agentic development is worth exploring.

---

## Installing

### Cursor

Drop into `.cursor/` in your project or workspace:

```
.cursor/
  rules/
    baseline-coding-standard.mdc
    engineering-behavior.mdc
    python-backend.mdc
  skills/
    architecture-and-api/
    backend-pre-merge-reviewer/
    ...
```

### Claude Code

Skills go in `.claude/skills/` — each folder contains a `SKILL.md` that Claude Code loads automatically when invoked. Rules go inline in `CLAUDE.md` (convert `.mdc` content to prose).

```
.claude/
  skills/
    implement/
    backend-pre-merge-reviewer/
    ...
CLAUDE.md   ← paste rule content here
```

Use the install prompt in `README.md` to have the agent handle the conversion automatically.

---

## What's in Here

```
rules/
  baseline-coding-standard.mdc   ← always on
  engineering-behavior.mdc       ← always on
  python-backend.mdc             ← always on for Python files
skills/
  architecture-and-api/
  backend-pre-merge-reviewer/
  caveman/
  clean-code-reviewer/
  debugging-systematic/
  design-to-build/
  frontend-pre-merge-reviewer/
  frontend-visual-design/
  grilling/
  grill-with-docs/
  implement/
  improve-codebase-architecture/
  post-rebase-verify/
  python-top-down/
  python-type-discipline/
  readme-writer/
  review-loop/
  security-auditor/
  stop-slop/
  systems-design/
  tdd/
  technical-docs-writer/
  ux-product/
  write-a-skill/
```

---

## Rules

Rules are always-on constraints. You don't invoke them. They load automatically
and shape how the agent writes code in every session.

### `baseline-coding-standard`
Universal naming, function design, error handling, and comment standards.
Language-agnostic. Enforces things like: no `x`/`tmp`/`data2` variable names,
one responsibility per function, no silent exception swallowing, comments explain
why not what.

### `engineering-behavior`
Defines how the agent behaves as an engineer: confirms requirements before writing
code, proposes architecture before implementing, thinks in trade-offs, and doesn't
over-engineer. The meta-rule that governs agent conduct across all tasks.

### `python-backend`
Python-specific standards: typing discipline, backend patterns, idiomatic Python
conventions. Activates automatically on `.py` files. Included since they frequently develop in Python. 
You can create one for any specific language

---

## Skills

Skills are on demand. They activate when you use a trigger phrase in chat, or you
can invoke them manually with `/skill-name`.

---

## Choosing the Right Workflow

**"I have a spec and need to build it"** → `implement`
Step-by-step: confirm context, search best practices (with dissenting sources), lock architecture, TDD slices, review loop, explicit pause for your sign-off, then ship.

**"I'm starting from scratch and don't know what to build yet"** → `design-to-build`
Blank-slate: structured grilling session, creates CONTEXT.md + KNOWLEDGEBASE.md + architecture plan, then phases into TDD build with review gates.

Use `design-to-build` to figure out what to build. Use `implement` to build it.

---

### `implement`
**What it does:** Full implementation loop from spec to ship. Confirms context, searches best practices with cited sources (and actively looks for dissenting perspectives to surface real tradeoffs), locks architecture before any code is written, builds in TDD vertical slices with code quality constraints, runs review loop, pauses for your sign-off, then ships.

**When to use:** Any time you have a clear requirement and the feature touches more than one layer or requires a design decision. The default for non-trivial work.

**Trigger phrases:** "implement this", "build this", "let's implement", "start the implementation loop"

---

### `design-to-build`
**What it does:** End-to-end workflow for blank-slate work — structured design grilling, explicit decision capture, `CONTEXT.md` + `KNOWLEDGEBASE.md` memory docs, phased implementation with `tdd`, then review gates before shipping.

**When to use:** Starting a new feature from scratch, significant refactor, or anything that requires design decisions before a single line of code is written. Use this before `implement` when you don't yet know what to build.

**Trigger phrases:** "design this", "let's design and build", "start from scratch", "design-to-build"

---

### `systems-design`
**What it does:** Walks through a design problem systematically — constraints,
trade-offs, scalability, failure modes — before any code is written.

**When to use:** Starting a new feature, designing an API, making a significant
architectural change, or any time the right approach isn't obvious yet.

**Trigger phrases:** "design this", "help me think through", "how should I architect",
"what's the best approach for"

---

### `architecture-and-api`
**What it does:** Enforces safe API and module boundary design — contract stability,
versioning, payload hygiene, interface changes that won't break callers.

**When to use:** After systems-design, when translating a design into actual API
contracts, endpoints, or module interfaces.

**Trigger phrases:** "design this API", "review this interface", "API boundary",
"module contract"

> **Use in sequence:** `systems-design` first for the big picture, then
> `architecture-and-api` to lock the contracts.

---

### `tdd` (assisted by Matt Pocock, edited by myself)
**What it does:** Drives feature development and bug fixes with a red-green-refactor
loop. One vertical slice at a time — one test, one implementation, repeat. Tests
verify behavior through public interfaces, not implementation details.

**When to use:** Building any non-trivial feature or fixing a bug where locking
down behavior matters. Called automatically by `implement`.

**Trigger phrases:** "build this with TDD", "test-first", "red green refactor",
"write tests for this"

---

### `debugging-systematic`
**What it does:** Enforces a structured diagnosis process — state the symptom,
separate facts from assumptions, build a feedback loop, trace the data flow, form
ranked hypotheses, confirm before fixing. Prevents jumping straight to a fix
before understanding the cause.

**When to use:** Any non-trivial bug. Especially silent failures, data not
persisting, state stuck in wrong value, or issues that survive a code change.

**Trigger phrases:** "debug this", "diagnose this", "something is broken",
"this is failing", "figure out why"

---

### `backend-pre-merge-reviewer`
**What it does:** Parallel modular backend review — dispatches independent review modules simultaneously, each gated on a signal in the diff (async/messaging patterns → messaging module; `synchronized`/`volatile` → concurrency module; migration files → DB migrations module; new endpoints → API contracts module). Always runs correctness, error handling, security, and style. Falls back to targeted file reads for large diffs. Finishes with an interactive Q&A step and an explicit evaluation of whether acceptance/integration tests are needed. Produces a structured report with MUST FIX / SHOULD FIX / NIT tiering.

**When to use:** Before merging any backend change that touches auth, data access,
API contracts, or business logic. Called automatically by `implement`.

**Trigger phrases:** "review this", "pre-merge check", "backend review",
"check before merging"

---

### `frontend-pre-merge-reviewer`
**What it does:** Staff-level frontend merge gate for user-facing UI: traces props,
state, effects, rendering, and realistic breakpoints (navigation, loading, races,
a11y). Pair with **`ux-product`** so implementation matches agreed flows and states.
Uses **web search** on the project's framework stack to calibrate common pitfalls
against the actual diff — not generic lectures.

**When to use:** Before merging any meaningful UI, client routing, or interaction
change; use with `ux-product` when product intent was explicitly decided first.

**Trigger phrases:** "frontend review", "pre-merge check UI", "review my React
change", "check this frontend PR", "UX implementation review"

---

### `clean-code-reviewer`
**What it does:** Reviews code for readability, maintainability, and best practices.
Checks naming, function design, complexity, dead code, comment quality, and
structure. Web-searches current best practices for the detected language before
reviewing. Issues a final verdict.

**When to use:** When you want a maintainability pass independent of security or
backend risk.

**Trigger phrases:** "review this code", "cleanliness check", "code quality review",
"is this clean", "maintainability review"

> **Review lane guide:**
> - Maintainability → `clean-code-reviewer`
> - Backend risk → `backend-pre-merge-reviewer`
> - Security → `security-auditor`
> Run these as separate passes, not all at once.

---

### `post-rebase-verify`
**What it does:** 6-step checklist to run after any rebase that required manual conflict resolution: diff stat sanity check, grep for intentional additions you named upfront (TODOs, constants, annotations), interface/implementation parity, structured file consistency (package manifests, DI configs, lock files), conflict marker scan, final diff review. Catches silent drops — missing methods, duplicate entries, lost TODOs — that only surface when a reviewer asks or a build fails.

**When to use:** After any rebase with manual conflict resolution, before re-requesting review.

**Trigger phrases:** "post-rebase check", "verify rebase", "check after rebasing", "did anything get dropped"

---

### `review-loop`
**What it does:** Iterates on AI reviewer feedback until zero open comments or the 5-cycle limit is hit. Classifies each comment by type (correctness, test-coverage, simplification, type-design, documentation, style, security) and scores fix confidence 0–100. High confidence (≥70) → fix directly; low confidence → reply with reasoning first. Detects flip-flops (same pattern re-commented in a later cycle) and surfaces them to you instead of reverting. Adds preemptive intent comments when making non-obvious fixes. Maintains a learned-patterns file to avoid repeating confirmed false positives across cycles.

**When to use:** After opening a PR and receiving AI review comments. Run this instead of manually addressing each comment.

**Trigger phrases:** "loop on review comments", "address review", "iterate on PR feedback", "keep going until reviewer is happy"

---

### `security-auditor`
**What it does:** Adversarial security audit with an infrastructure-first phase ordering. Starts with an attack surface census (public endpoints, admin routes, file uploads, background jobs, CI workflows, container configs), then reviews secrets → supply chain → CI/CD → infrastructure → application code → OWASP. After confirming a finding, auto-searches for the same pattern across all files. Includes incident response playbooks for leaked secrets. Every finding requires ≥8/10 confidence and a concrete exploit scenario — specific inputs, attacker position, what breaks. Can save reports to `.security-reports/` for trend tracking (Resolved/Persistent/New) across runs.

**When to use:** Before shipping anything security-sensitive, after adding auth,
when integrating third-party services, or as a dedicated security sprint. Called
automatically by `implement`.

**Trigger phrases:** "security audit", "audit this codebase", "find vulnerabilities",
"adversarial review", "security check"

---

### `python-type-discipline`
**What it does:** Enforces strict typing hygiene in Python — function signatures,
Pydantic models, return types, typed error handling.

**When to use:** When refactoring Python interfaces, adding Pydantic models,
or doing a typing pass on existing code.

**Trigger phrases:** "type this", "add types", "typing pass", "fix the types"

---

### `python-top-down`
**What it does:** Enforces top-down Python module structure so readers see public API
and high-level orchestration first, then private helpers and low-level utilities.

**When to use:** Writing, reviewing, or refactoring Python modules.

**Trigger phrases:** "top-down structure", "module ordering", "python readability",
"why is this function hard to find"

---

### `grilling`
**What it does:** One-question-at-a-time decision grilling. Asks a single targeted
question, provides a recommended answer, waits, pushes back on risky answers, then
moves to the next. Used as a sub-skill inside `grill-with-docs` and
`improve-codebase-architecture`.

**When to use:** Standalone when you want to stress-test a specific decision or
assumption without the full docs-grounding context of `grill-with-docs`.

**Trigger phrases:** "grill me on this decision", "one question at a time", "stress-test this"

---

### `grill-with-docs` (by Matt Pocock)
**What it does:** Stress-tests a plan against the repo's domain model — challenges
terminology against `CONTEXT.md`, cross-references decisions against ADRs, updates
docs inline as decisions land. Delegates the grilling loop to the `grilling` skill.

**When to use:** Before starting any significant task when the project already has
(or should have) shared context docs. Use before `systems-design`.

**Trigger phrases:** "grill me on this", "grill with docs", "stress-test this plan",
"challenge my approach", "what am I missing"

---

### `ux-product`
**What it does:** Frames work in terms of user goals and product outcomes before
implementation. Clarifies user state, flow, and intent.

**When to use:** Frontend feature work, product decision-making, or any time
you're building something user-facing and want to think through the experience
before touching code.

**Trigger phrases:** "think through the UX", "product framing", "user flow",
"what should the user experience be"

---

### `improve-codebase-architecture` (by Matt Pocock, updated)
**What it does:** Finds deepening opportunities — refactors that turn shallow modules
into deep ones. YAGNI-first: starts with git log hot spots, not a broad scan.
Generates an HTML report with recommendation cards. Flags ADR conflicts. Delegates
the decision loop to the `grilling` skill.

**When to use:** When a codebase is becoming hard to navigate, when modules feel too
small and too coupled, or periodically as a code health practice.

**Trigger phrases:** "improve the architecture", "find refactoring opportunities", "codebase health check", "this feels like a ball of mud"

---

### `write-a-skill` (by Matt Pocock)
**What it does:** Helps you build new SKILL.md files using a structured template.

**When to use:** When you've identified a recurring workflow that doesn't have a skill yet.

**Trigger phrases:** "create a skill for", "write a new skill", "encode this workflow", "build a skill that"

---

### `caveman` (by Matt Pocock)
**What it does:** Switches the assistant into ultra-compressed communication mode:
short, direct responses with technical substance preserved and filler removed.

**When to use:** When you want lower token usage, tight back-and-forth loops,
or terse execution updates.

**Trigger phrases:** "caveman mode", "talk like caveman", "use caveman",
"less tokens", "be brief"

---

### `readme-writer`
**What it does:** Writes or rewrites project READMEs in a direct engineering
voice — no AI filler, no justification language, no teacher voice.

**When to use:** Creating or updating a project README.

**Trigger phrases:** "write a readme", "update the readme", "document this project"

---

### `technical-docs-writer`
**What it does:** Writes module-level technical documentation as a navigable
system — top-level README describes full architecture, each module README
describes itself and leads to the next.

**When to use:** Documenting a full codebase, writing module READMEs, or
creating architecture documentation.

**Trigger phrases:** "document this codebase", "write technical docs",
"document this module", "write architecture docs"

> **Docs lane guide:**
> - Project README → `readme-writer`
> - System/module documentation → `technical-docs-writer`

---

## Sample Workflows

### Non-trivial feature (spec in hand)

```
1. implement         → context confirm → search → architecture lock → TDD → review → your sign-off → ship
2. review-loop       → after opening PR, iterate on AI review comments until clean
3. post-rebase-verify → after any rebase with conflicts, before re-requesting review
```

### Blank-slate feature (no spec yet)

```
1. grill-with-docs   → align on language and existing constraints first
2. design-to-build   → structured grilling → docs → phased TDD → review → ship
```

### Backend API Feature (Python)

```
1. implement         → full loop; apply python-type-discipline where signatures/models change
```

### Frontend Feature + Backend Integration

```
1. ux-product        → clarify user goal and flow before implementation
2. implement         → full loop; use frontend-pre-merge-reviewer + backend-pre-merge-reviewer in review phase
```

### Non-Trivial Bug

```
1. debugging-systematic → facts vs assumptions, ranked hypotheses, confirm cause
2. tdd               → regression test before the fix
3. backend-pre-merge-reviewer → if the bug touched a critical backend path
```

### Security Audit Sprint

```
1. security-auditor  → run as the primary process, multi-pass
2. After audit closes: route remediation through implement
```

### Documentation Pass

```
- System/module docs:   technical-docs-writer
- Project README:       readme-writer
- Run one per task, not both simultaneously
```

---

## Tips

**Don't stack review skills simultaneously.** `clean-code-reviewer`,
`backend-pre-merge-reviewer`, and `security-auditor` each have a specific
lens. Running all three at once produces noise. Pick one per pass by objective.

**Scope your asks.** If you ask the agent to "analyze the codebase," it will
scan broadly and miss things. Ask it to analyze a specific module.

**`grill-with-docs` before `systems-design`.** `grill-with-docs` challenges
whether you're solving the right problem. `systems-design` solves the problem
correctly. They do different things.

**Write module READMEs as you build.** Use `technical-docs-writer` after
finishing each module, not at the end of the project.
