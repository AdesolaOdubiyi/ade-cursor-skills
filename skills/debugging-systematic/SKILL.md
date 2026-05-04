---
name: debugging-systematic
description: |
  Structured diagnosis loop for hard bugs, silent failures, and performance
  regressions. Activate when user says: debug this, diagnose this, something
  is broken, this is throwing, this is failing, data not persisting, status
  stuck, bug report, performance regression, this isn't working.
---

# Systematic Debugging

A discipline for hard bugs. Skip phases only when explicitly justified.

---

## Phase 1 — State the symptom precisely

One sentence: exactly what is observed vs what is expected.
This is the target. Every subsequent phase must stay anchored to it.
Wrong bug = wrong fix.

---

## Phase 2 — Separate facts from assumptions

List what is **confirmed** and what is **assumed**.
Do not treat assumptions as facts. Do not hypothesise yet.

---

## Phase 3 — Build a feedback loop

**This is the skill.** If you have a fast, deterministic, runnable pass/fail
signal for the bug, you will find the cause. If you don't, no amount of
reading code will save you. Spend disproportionate effort here.

Try these in order:

1. Failing test at whatever seam reaches the bug
2. CLI invocation with a fixture input, diffing output against a known-good snapshot
3. Curl / HTTP script against a running dev server
4. Replay a captured trace — save a real payload to disk, replay in isolation
5. Throwaway harness — minimal subset of the system, single function call
6. Headless browser script if the bug is UI-side
7. Property / fuzz loop if the bug is "sometimes wrong output" — run 1000 inputs
8. Bisection harness if the bug appeared between two known states

Once you have a loop, improve it:
- Can it run faster? (Narrow scope, skip unrelated init)
- Is the signal sharp? (Assert on the specific symptom, not "didn't crash")
- Is it deterministic? (Pin time, seed RNG, isolate filesystem, freeze network)

**Non-deterministic bugs:** the goal is not a clean repro but a higher
reproduction rate. Loop 100×, add stress, narrow timing windows.
A 50%-flake is debuggable. 1% is not — raise the rate first.

**If you cannot build a loop:** stop and say so. List what you tried.
Ask the user for: a reproducing environment, a captured artifact
(log dump, HAR file, screen recording), or permission to add temporary
instrumentation. Do not proceed to Phase 4 without a loop.

---

## Phase 4 — Trace the data flow

- Find the last confirmed good state
- Find the first confirmed bad state
- The bug lives between those two points

Narrow the search space before hypothesising.

---

## Phase 5 — Form a minimal hypothesis list

Generate 3 ranked hypotheses before testing any of them.
Single-hypothesis generation anchors on the first plausible idea.

Each hypothesis must be falsifiable:
> "If X is the cause, then changing Y will make the bug disappear /
> changing Z will make it worse."

If you cannot state the prediction, the hypothesis is a hunch — discard or sharpen it.

Show the ranked list to the user before testing. They often have domain
knowledge that re-ranks instantly. Don't block on it — proceed with your
ranking if the user is unavailable.

---

## Phase 6 — Instrument to confirm

Each probe maps to a specific prediction from Phase 5.
Change one variable at a time.

- Debugger / REPL inspection first if the environment supports it
- Targeted logs at the boundaries that distinguish hypotheses
- Never log everything and grep

Tag every debug log with a unique prefix: `[DBG-xxxx]`.
Cleanup becomes a single grep. Untagged logs survive; tagged logs die.

**Performance regressions:** logs are usually wrong here. Establish a
baseline measurement first (timing harness, profiler, query plan),
then bisect. Measure before fixing.

Confirm the hypothesis with evidence before touching production logic.

---

## Phase 7 — Fix root cause, not symptom

Write the regression test **before** the fix — but only if a correct seam exists.

A correct seam is one where the test exercises the real bug pattern at the
actual call site. If the only available seam is too shallow, a test there
gives false confidence. If no correct seam exists, that is itself a finding —
note it and flag the architectural gap.

If a correct seam exists:
1. Turn the minimised repro into a failing test
2. Watch it fail
3. Apply the fix
4. Watch it pass
5. Re-run the Phase 3 loop against the original scenario

---

## Phase 8 — Cleanup and close

- [ ] Original repro no longer reproduces
- [ ] Regression test passes, or absence of seam is documented
- [ ] All `[DBG-...]` instrumentation removed
- [ ] Throwaway harnesses deleted
- [ ] The correct hypothesis is stated in the commit message

Then ask: what would have prevented this bug? If the answer involves
architectural change, make that recommendation after the fix is in —
you have more information now than when you started.