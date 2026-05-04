---
name: technical-docs-writer
description: |
  Write or update technical documentation for a codebase or module.
  Activate when user says: document this codebase, write technical docs,
  document this module, write a module readme, document the architecture,
  write docs for this service, create architecture docs, document this system.
---

# Technical Documentation Writer

## Purpose
An engineer with no internet, no AI, and no colleagues should be able to read
from the top-level README downward and understand the system well enough to
make changes confidently. Every README is a chapter. The system is the book.

This skill produces two types of documentation:
- **Top-level README** — full system architecture, module map, system-wide decisions
- **Module README** — module responsibility, internals, decisions, references

Determine which type is needed before writing. If documenting a full codebase,
start at the top level and work downward.

---

## Step 1 — Scan Before Writing
Before writing any documentation:
1. Read the full file tree
2. Identify all meaningful modules (ignore trivial folders — `utils/`, `constants/`,
   `types/` unless they contain non-obvious logic)
3. Trace the primary data or request flow from entry point to output
4. Identify any non-trivial architectural decisions visible from structure alone
   (unusual patterns, non-standard conventions, unexpected dependencies)

Do not write until you understand how the pieces connect.

---

## Top-Level README

### When to write it
When documenting a full system, service, or repository for the first time,
or when the existing top-level README doesn't describe architecture or modules.

### Structure
```markdown
# Project Name
One sentence: what the system does and who/what it serves.

## Architecture
[Mermaid diagram — all major modules and the flow between them]
[2-3 sentences of prose: describe the primary data or request lifecycle
at the highest level of abstraction. No implementation detail here.]

## Modules
[Ordered list — most foundational first, then dependents]
1. `module-path/` — one sentence: what it owns
2. `module-path/` — one sentence: what it owns

## Design Decisions
[System-wide non-trivial choices only. See format below.]

## References
[Ordered by relevance. See format below.]
```

### Diagram guidance
- Use Mermaid by default — renders in GitHub, GitLab, and Cursor
- Fall back to ASCII only if the target environment is unknown or a terminal
- Diagram scope: all major modules and their directional relationships
- Do not diagram file-level detail at the top level — that belongs in module READMEs
- Keep it readable. If the diagram exceeds ~15 nodes, split into two: one for
  system topology, one for data flow

---

## Module README

### When to write it
Write a module README when the module:
- Has non-trivial internal structure
- Contains design decisions an engineer would ask "why?" about
- Could be worked in independently from its parent
- Has dependencies on other modules that aren't obvious from imports alone

Do NOT write a module README when:
- The module is a utility folder, constants file, or type definitions with no logic
- Everything in it is self-explanatory from filenames and standard conventions
- A one-sentence description in the parent's Sub-modules section is sufficient

For trivial modules, write one sentence in the parent README. That is the documentation.

### Structure
```markdown
# module-name/
One sentence: this module's single responsibility within the system.

## Overview
[Prose — 1-2 paragraphs. This is the story section.
Explain what this module does, why it exists as a separate module,
and where it sits in the request or data lifecycle.
Write as if explaining to a new engineer on the team — precise but not terse.
Answer: what does this do, why is it its own thing, what should I understand
before I touch it.]

## Structure
[File tree with annotations — non-obvious files only.
Standard files (index.ts, types.ts, README.md) need no annotation.
Annotate anything where the name alone doesn't explain the purpose.]

## Design Decisions
[Non-trivial choices scoped to this module. See format below.]

## Sub-modules
[Only if sub-modules exist. Ordered list — most foundational first.]
[Trivial sub-modules get one sentence here, not their own README.]
[Non-trivial sub-modules get a link to their own README.]

## References
[Ordered by relevance. See format below.]
```

---

## Design Decisions Format

Each entry answers: what was decided, why, and what breaks if someone changes
it without knowing.

Use this format:
```markdown
**[Decision title]**
[What was chosen and what was rejected or what the alternative was.]
[Why — the constraint, the tradeoff, or the failure mode that drove the decision.]
[What breaks or degrades if this is changed without understanding it.]
```

Example:
```markdown
**Redis for session storage instead of the primary DB**
Session reads happen on every authenticated request. At load, DB round-trips
at that frequency caused p99 latency spikes. Changing this requires updating
the auth middleware and the invalidation logic in `services/auth/`.
```

Depth of explanation is proportional to the complexity of the tradeoff.
A one-sentence decision needs one sentence. A genuinely complex tradeoff
with cascading implications warrants a short paragraph. Use judgment.

Non-trivial means: a new engineer would ask "why did they do it this way?"
If the answer is "that's just standard practice," it is trivial — omit it.

---

## References Format

Ordered by relevance — most important to least. Each entry has a description
explaining why it is referenced, not just what it is.

```markdown
1. `services/auth/` — token validation this module calls on every request
2. `lib/db/` — shared query helpers; see connection pooling config there
3. [Stripe webhook event docs](url) — canonical shape of events this module ingests
4. [RFC 7519 — JWT](url) — token format reference for the signing logic
```

For internal module references, use relative paths.
For external references, use full URLs with descriptive link text.
Order: internal dependencies first, then external docs, then background reading.

---

## Banned Patterns
The same writing rules from the readme-writer skill apply here, plus:

**Restating the code** — do not describe what a function does if the name says it
- No: validateToken() validates the token
- Yes: Omit it, or explain why validation works the way it does

**Duplicating parent README content** — each doc owns its scope
- If the top-level README explains the architecture, modules do not repeat it
- If a module README explains a decision, sub-modules do not re-explain it

**Documenting standard conventions** — if it is standard for the framework or language, skip it
- No: "This folder contains React components used throughout the app"
- Yes: Only document what deviates from or extends the convention

**Forward references without substance** — "see X for more details" with no
indication of what the reader will find there or why they need it

---

## The Recursion Rule

Ask for each sub-module: could an engineer work here without understanding
the parent? Does it have its own decisions? Its own non-obvious structure?

- Yes to any → it gets its own README, following this same skill
- No → one sentence in the parent's Sub-modules section

There is no fixed depth limit. Depth follows complexity.

---

## Final Check

Before finishing any README:
1. Can an engineer trace the full flow from the top-level README to this module
   without any gaps?
2. Does every Design Decision entry explain what breaks if ignored?
3. Does the References section cover everything an engineer needs to work here?
4. Is there any content that duplicates a parent or child README?
5. Is there any sentence that defends, justifies, or congratulates the code?

Items 4 and 5 — delete. Items 1 through 3 — fix the gap.