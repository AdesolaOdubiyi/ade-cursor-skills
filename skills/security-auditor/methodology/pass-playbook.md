# Pass Playbook

## What Each Pass Is Trying to Do

### Phase 1 — Adversarial Discovery
Objective: find every vulnerability that exists with supporting evidence.
Approach: assume malicious authorship. Treat every security-sensitive path as
suspicious until the code proves otherwise.
Entry points: start from all external inputs and trace inward. Start from all
auth checks and look for bypasses. Start from all data writes and look for
injection paths.

### Phase 2 — Self-Challenge
Two simultaneous objectives:

**Objective A — Disprove Uncertain findings**
For each Uncertain finding from Phase 1:
- Look for code that would prevent the exploit from being reachable
- Look for validation, sanitization, or auth checks that would block it
- If counter-evidence is found: mark Disproved, note the evidence, do not delete
- If no counter-evidence is found after thorough search: upgrade to Likely

**Objective B — Find what Phase 1 missed**
- Approach from entry points not used in Phase 1
- Look for interaction effects between modules (finding A + finding B = finding C)
- Look for time-of-check/time-of-use patterns across files
- Look for second-order effects of findings already confirmed

### Phase 3 — Final Pass
Same as Phase 2. This is the hard ceiling.
After this pass the session closes regardless of what remains.

## Targeted Questions Protocol
After each pass, before proceeding:
1. List every finding where product behavior affects classification
2. List every code path where you could not determine if behavior was intentional
3. Ask the user directly — one question per ambiguity
4. Record every question and answer in the Q&A Log in the report
5. Revise finding classifications based on answers before the next pass begins

Never guess. Never assume. If you do not know whether a behavior is intentional,
you must ask before classifying it.
