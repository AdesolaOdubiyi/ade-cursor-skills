# Context consolidation checklist

Use at the end of every build-with-confidence cycle.

## Before you edit

- [ ] What shipped this build?
- [ ] What blockers were discovered (with evidence: test name, URL, error)?
- [ ] Any new ADR-worthy decisions?
- [ ] What is still explicitly **not** in scope?

## Semantic updates (replace stale)

- [ ] **STATUS.md** — update checkmarks; remove completed "next" items
- [ ] **CONTEXT.md** — only stable product truths; no session chatter
- [ ] **DECISIONS.md** — new ADR if decision must not be re-litigated
- [ ] **RUNBOOK.md** — commands/operators changed this build

## Episodic record (append OK)

- [ ] **docs/build-log/YYYY-MM-DD.md** — short bullet: what ran, what passed/failed
- [ ] Runtime logs (e.g. `batch_results.jsonl`) — keep raw outcomes; don't paste into CONTEXT

## Handoff (if work continues)

- [ ] **docs/HANDOFF.md** — goals, % progress, prioritized next tasks, key paths, commands
- [ ] Fresh agent should not need prior chat transcript

## Pass 2 — completeness audit (after Pass 1 edits)

- [ ] Launch readonly completeness subagent (see `../completeness-audit-prompt.md`)
- [ ] Reconcile all high/medium findings
- [ ] Re-audit until `CLEAN: yes` (max 3 rounds)

## Quality bar

- [ ] No duplicate facts across CONTEXT / STATUS / HANDOFF
- [ ] Stale "planned" language removed where feature exists
- [ ] Contradictions resolved (recency for status, ADR for intent)
