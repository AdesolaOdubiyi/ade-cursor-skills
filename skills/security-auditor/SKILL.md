---
name: security-auditor
disable-model-invocation: false
description: |
  Adversarial multi-pass security audit of any codebase.
  Activate when user says: security audit, audit this codebase, security review,
  find vulnerabilities, audit this, security check, run the auditor,
  check for vulnerabilities, adversarial review.
---

# Security Auditor

## What This Skill Does
Conducts a structured, evidence-based, adversarial security audit across multiple
passes. Every finding requires code-level evidence and a confidence classification.
Findings without both are discarded — not reported as Uncertain.

This skill assumes the code was written by an opposing agent with deliberate intent
to introduce undetectable vulnerabilities. Every security-sensitive path is treated
as potentially malicious until evidence suggests otherwise.

## How to Run a Session

Follow these phases in order. Do not skip phases. Do not merge phases.

### Phase 0 — Scope Map
Before writing any findings:
1. Scan the full file and directory tree
2. Identify every attack surface (see methodology/threat-model.md)
3. Classify each surface as Primary (full adversarial pass) or Secondary (lighter pass)
   using the layer weight table in methodology/threat-model.md
4. List every area where product context is needed before you can assess correctly
5. Present the scope map to the user
6. Ask every targeted question you have before proceeding — never assume product behavior
7. Do not begin Phase 1 until the user approves the scope map and answers your questions

### Phase 1 — First Adversarial Pass
1. Review all Primary surfaces using checklists/application-code.md
2. Review all Secondary surfaces using checklists/infra-and-config.md
3. For every suspected finding, apply the evidence gate in report/finding-format.md
   before recording it — if the finding cannot satisfy all three fields, discard it
4. Flag any finding matching patterns in checklists/suspicious-intent.md separately
5. When the pass is complete, write the full report using report/template.md
6. Ask the user targeted questions about anything you could not determine from
   code alone — intentional bypasses, product-specific flows, expected behaviors
   that look like bugs. Record every question and answer in the Q&A Log.
7. Wait for the user before proceeding to Phase 2

### Phase 2 — Self-Challenge Pass
Run both objectives simultaneously:
1. Attempt to disprove every Uncertain finding from Phase 1 by finding counter-evidence
   — if disproved, mark the finding Disproved with a note, do not delete it
   — if confirmed, upgrade its confidence classification
2. Approach the codebase from new entry points to find what Phase 1 missed
   — new findings follow the same evidence gate as Phase 1

After Phase 2, apply the stopping rules in methodology/stopping-rules.md.
If the stopping condition is met, produce the final report and close the session.
If not, proceed to Phase 3.

Ask the user targeted questions again before proceeding to Phase 3.
Wait for the user.

### Phase 3 — Final Pass (Hard Ceiling)
Run the same process as Phase 2.
After Phase 3, stop regardless of findings remaining.
Mark all Uncertain findings with unresolvable evidence as explicitly unresolved.
Produce the final report. Close the session.
Do not offer to run another pass. The session is closed.

## Core Rules
- Never report a finding you cannot cite at function level
- Never assume product behavior — ask the user
- Never escalate severity without evidence
- Suspicious intent escalates severity by one tier automatically
- The report is cumulative and versioned — never overwrite previous pass findings
- Resolved findings are struck through, not deleted
