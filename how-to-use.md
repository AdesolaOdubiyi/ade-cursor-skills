# Cursor Skills & Rules — How to Use

A personal toolkit of Cursor rules and agent skills built for production-quality
engineering work. Covers the full stack with a backend-heavy bias.

Feel free to drop these into your own `.cursor/` directory and adapt them to your
project. The rules apply automatically; the skills activate by trigger phrase or
manual invocation.

## Credits

Several skills in this setup are authored by [Matt Pocock](https://github.com/mattpocock/skills)
and used here with attribution. These include `improve-codebase-architecture`,
`grill-with-docs`, `write-a-skill`, and `caveman`. His full skills library and further information he provides about agentic development is worth exploring.

---

## What's in Here

```
.cursor/
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
    grill-with-docs/
    improve-codebase-architecture/
    python-top-down/
    python-type-discipline/
    readme-writer/
    security-auditor/
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

## Start Here: `design-to-build`

Before reading into individual skills further, I'd strongly recommend starting with
`design-to-build`. In my experience, it is the highest-leverage skill in this set for
non-trivial architectural work.

It forces the agent through a specified workflow: structured design grilling, explicit
decision capture, `CONTEXT.md` + `KNOWLEDGEBASE.md` memory docs, phased implementation
with `/tdd`, then clean-code, pre-merge, and security gates before shipping. It
orchestrates multiple skills so architecture, execution, and review stay aligned.

Use this first when the work is bigger than a quick patch.

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
down behavior matters.

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
**What it does:** Rigorous pre-merge review of backend code — traces the full
request lifecycle, checks auth gaps, race conditions, partial failures, contract
drift, secret handling, and error handling. Assumes the code was AI-generated and
reviews accordingly. Produces a structured findings report with a final verdict.

**When to use:** Before merging any backend change that touches auth, data access,
API contracts, or business logic.

**Trigger phrases:** "review this", "pre-merge check", "backend review",
"check before merging"

---

### `frontend-pre-merge-reviewer`
**What it does:** Staff-level frontend merge gate for user-facing UI: traces props,
state, effects, rendering, and realistic breakpoints (navigation, loading, races,
a11y). Pair with **`ux-product`** so implementation matches agreed flows and states.
Uses **web search** on the project’s framework stack (often TypeScript/React) to
calibrate common pitfalls and failure modes against the actual diff — not generic
lectures.

**When to use:** Before merging any meaningful UI, client routing, or interaction
change; use with `ux-product` when product intent was explicitly decided first.

**Trigger phrases:** "frontend review", "pre-merge check UI", "review my React
change", "check this frontend PR", "UX implementation review"

---

### `clean-code-reviewer`
**What it does:** Reviews code for readability, maintainability, and best practices.
Checks naming, function design, complexity, dead code, comment quality, and
structure. Web-searches current best practices for the detected language before
reviewing. Produces violations only — no praise, no suggestions for already-clean
code. Issues a final verdict.

**When to use:** When you want a maintainability pass independent of security or
backend risk. Good for frontend code, utilities, and any non-backend logic.

**Trigger phrases:** "review this code", "cleanliness check", "code quality review",
"is this clean", "maintainability review"

> **Review lane guide:**
> - Maintainability → `clean-code-reviewer`
> - Backend risk → `backend-pre-merge-reviewer`
> - Security → `security-auditor`
> Run these as separate passes, not all at once.

---

### `security-auditor`
**What it does:** Multi-pass adversarial security audit. Assumes the code was
written by an opposing agent with intent to introduce undetectable vulnerabilities.
Runs until no further evidence-backed findings can be found. Every finding requires
function-level citation and a confidence score — no hunches get reported. Produces
a versioned cumulative report across passes.

**When to use:** Before shipping anything security-sensitive, after adding auth,
when integrating third-party services, or as a dedicated security sprint.

**Trigger phrases:** "security audit", "audit this codebase", "find vulnerabilities",
"adversarial review", "security check"

---

### `python-type-discipline`
**What it does:** Enforces strict typing hygiene in Python — function signatures,
Pydantic models, return types, typed error handling. Focused on interface and
model design rather than broad backend conventions.

**When to use:** When refactoring Python interfaces, adding Pydantic models,
or doing a typing pass on existing code.

**Trigger phrases:** "type this", "add types", "typing pass", "fix the types"

---

### `python-top-down`
**What it does:** Enforces top-down Python module structure so readers see public API
and high-level orchestration first, then private helpers and low-level utilities.
Useful for readability and maintainability in medium/large modules.

**When to use:** Writing, reviewing, or refactoring Python modules (especially when a
function is hard to find, or module ordering is confusing).

**Trigger phrases:** "top-down structure", "module ordering", "python readability",
"why is this function hard to find"

---

### `grill-with-docs` (by Matt Pocock)
**What it does:** Same relentless interview style as a classic "grill me" session,
but grounded in your repo: stress-tests the plan against the domain model,
sharpens terminology, and updates `CONTEXT.md` / `docs/adr/` as decisions land.

**When to use:** Before starting any significant task when the project already has
(or should have) shared context docs. Especially useful before `systems-design`.

**Trigger phrases:** "grill me on this", "grill with docs", "stress-test this plan",
"challenge my approach", "what am I missing"

---

### `ux-product`
**What it does:** Frames work in terms of user goals and product outcomes before
implementation. Clarifies user state, flow, and intent. Useful for making sure
UI and product decisions are grounded in what the user actually needs.

**When to use:** Frontend feature work, product decision-making, or any time
you're building something user-facing and want to think through the experience
before touching code.

**Trigger phrases:** "think through the UX", "product framing", "user flow",
"what should the user experience be"

---

### `improve-codebase-architecture` (by Matt Pocock)
**What it does:** Finds deepening opportunities in a codebase — refactors that turn shallow modules into deep ones. Explores organically for architectural friction: tightly-coupled modules, untestable seams, shallow interfaces. Works best alongside a CONTEXT.md and docs/adr/ for domain context.

**When to use:** When a codebase is becoming hard to navigate, when modules feel too small and too coupled, or periodically as a code health practice. Matt Pocock recommends running it every few days.

**Trigger phrases:** "improve the architecture", "find refactoring opportunities", "codebase health check", "this feels like a ball of mud"

---

### `write-a-skill` (by Matt Pocock)
**What it does:** Helps you build new SKILL.md files using a structured template. Interviews you on the use case, generates the skill with correct frontmatter, and validates it follows the lean, token-efficient format.

**When to use:** When you've identified a recurring workflow that doesn't have a skill yet, or when you want to encode a pattern so you don't repeat the same setup instructions every session.

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
voice — no AI filler, no justification language, no teacher voice. Adapts
structure to project type (library, service, data pipeline, portfolio project).

**When to use:** Creating or updating a project README.

**Trigger phrases:** "write a readme", "update the readme", "document this project"

---

### `technical-docs-writer`
**What it does:** Writes module-level technical documentation as a navigable
system — top-level README describes full architecture, each module README
describes itself and leads to the next. Designed so an engineer with no other
resources can read from the top down and understand the system well enough to
make changes.

**When to use:** Documenting a full codebase, writing module READMEs, or
creating architecture documentation.

**Trigger phrases:** "document this codebase", "write technical docs",
"document this module", "write architecture docs"

> **Docs lane guide:**
> - Project README → `readme-writer`
> - System/module documentation → `technical-docs-writer`
> Don't run both at once unless doing a top-level README + module docs
> in separate explicit steps.

---

## Sample Workflows

These are the patterns that get the most out of this toolkit. Mix and match
based on what you're building.

---

### Backend API Feature (Python)

```
1. design-to-build   → run the full design → phased build → review workflow
2. During build phases: apply python-type-discipline where signatures/models change
3. In review phase: backend-pre-merge-reviewer (+ security-auditor if sensitive paths changed)
```

---

### Frontend Feature + Backend Integration

```
1. ux-product        → clarify user goal and flow before implementation
2. design-to-build   → execute full workflow (design docs, phased TDD, review gates)
3. In review phase: frontend-pre-merge-reviewer + backend-pre-merge-reviewer
4. Add security-auditor if auth, persistence, env vars, or external APIs changed
```

---

### Non-Trivial Bug

```
1. debugging-systematic → facts vs assumptions, ranked hypotheses, confirm cause
2. tdd               → regression test before the fix
3. backend-pre-merge-reviewer → if the bug touched a critical backend path
```

---

### Security Audit Sprint

```
1. security-auditor  → run as the primary process, multi-pass
   (keep other review skills off during the audit to avoid conflicting signals)
2. After audit closes: for non-trivial remediation, route work through design-to-build
   so fixes are designed, implemented in phases, and re-reviewed before ship
```

---

### Documentation Pass

```
- System/module docs:   technical-docs-writer
- Project README:       readme-writer
- Run one per task, not both simultaneously
```

---

### Starting a New Project

```
1. design-to-build   → establish CONTEXT + architecture plan + phased execution path
2. ux-product        → run during design/build if the project is user-facing
3. technical-docs-writer → document modules/architecture as phases complete
```

---

## Tips

**Don't stack review skills simultaneously.** `clean-code-reviewer`,
`backend-pre-merge-reviewer`, and `security-auditor` each have a specific
lens. Running all three at once produces noise. Pick one per pass by objective.

**Scope your asks.** If you ask the agent to "analyze the codebase," it will
scan broadly and miss things. Ask it to analyze a specific module. The more
scoped the ask, the deeper the agent goes.

**`grill-with-docs` before `systems-design`.** It sounds redundant but it isn't.
`grill-with-docs` challenges whether you're solving the right problem and aligns
language with the repo. `systems-design` solves the problem correctly. They do
different things.

**Write module READMEs as you build.** Use `technical-docs-writer` after
finishing each module, not at the end of the project. The agent will use those
READMEs as navigation aids in future sessions instead of re-scanning files
it's already seen.
