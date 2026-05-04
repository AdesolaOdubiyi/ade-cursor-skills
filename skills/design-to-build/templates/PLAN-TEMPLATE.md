# [Project Name] — Architecture Plan

Does not change after Design except to **tick phase checkboxes**. For day-to-day execution state, see `docs/PROGRESS.md`.

## Decisions

| ID | Decision |
|----|----------|
| D1 | [Exact agreed wording — no paraphrase] |
| D2 | [Exact agreed wording] |

## Target Structure

<!-- Directory tree with annotations: + add, ~ change, - remove, ? investigate -->

```
[annotated tree]
```

## Migration Phases

### Phase 0 — Housekeeping

**Rule:** No behavior changes. Diff may include renames, deletes, config-only edits, comments.

- [ ] [Housekeeping task]
- [ ] [Housekeeping task]

**Validation:** [How you prove zero behavior drift — tests green, diff review checklist, etc.]

### Phase 1 — [Name]

- [ ] [Task]
- [ ] [Task]

**Validation:** [Explicit pass/fail criteria]

**Tests required:**

- [Specific test case including edge case]
- [Specific test case]

Run tests multiple times — a single passing run is not confirmation.

### Phase 2 — [Name]

- [ ] [Task]

**Validation:** [Criteria]

**Tests required:**

- [Test case]

Run tests multiple times — a single passing run is not confirmation.

### Phase N — Cutover and Cleanup

- [ ] [Final cutover task]
- [ ] [Remove dead code / flags / migration scaffolding]

**Validation:** [Production-like verification]

**Tests required:**

- [Regression + edge]

Run tests multiple times — a single passing run is not confirmation.

## Testing Standards

| ID | Standard |
|----|----------|
| T1 | Follow `/tdd` for all new behavior |
| T2 | [Test framework / runner for this repo] |
| T3 | Edge cases required for DB/API/auth paths per phase |
| T4 | Run tests multiple times before ticking a phase — a single passing run is not confirmation |
| T5 | Tests assert through **public interfaces** only |

## Risk Register

| ID | Risk | Mitigation |
|----|------|------------|
| R1 | [Risk] | [Mitigation] |

## Future Work

| Item | Notes | Skill to use |
|------|-------|--------------|
| [Item] | [Notes] | e.g. `systems-design`, `security-auditor` |
