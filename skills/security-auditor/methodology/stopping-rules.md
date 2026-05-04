# Stopping Rules

## Automatic Stop Condition
After Phase 2 or Phase 3, check both conditions:

**Condition A — Diminishing returns**
New confirmed or likely findings in this pass < 20% of total confirmed or likely
findings from Phase 1.

**Condition B — Confidence floor**
No Uncertain findings remain that have citable evidence supporting them.
All remaining Uncertain findings are undecidable without external context
you do not have and the user cannot provide.

If both conditions are met: declare the session done, produce the final report,
close the session. Do not offer to run another pass.

## Hard Ceiling
Phase 3 is the absolute maximum. After Phase 3, stop regardless of whether
the stopping condition is met. Mark unresolved Uncertain findings explicitly.
Do not self-initiate a Phase 4 under any circumstances.

## Manual Override
The user can force an additional pass by explicitly asking for one.
This overrides the stopping condition but not the hard ceiling.
If the user asks for more passes after Phase 3, explain that the session
is closed and offer to start a new session instead.

## Hallucination Gate
This is the primary guardrail against fabricated findings.

A finding must satisfy all three before it is recorded:
1. Function-level citation — exact file path and function name
2. Exploit path — how it is actually reachable, not theoretically reachable
3. Confidence classification — with explicit reasoning for the classification

If any of the three cannot be satisfied, the finding is discarded entirely.
It is not recorded as Uncertain. It does not appear in the report.
Uncertain is not a placeholder for hunches. Uncertain means: real evidence
exists, but exploitability cannot be determined without more context.
