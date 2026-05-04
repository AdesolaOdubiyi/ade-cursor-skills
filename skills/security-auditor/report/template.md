# Security Audit Report

Project: {project name}
Session opened: {date}
Session status: In Progress / Closed

---

## Pass Summary

| Pass | New Findings | Confirmed | Likely | Uncertain | Disproved |
|------|-------------|-----------|--------|-----------|-----------|
| Phase 1 | | | | | — |
| Phase 2 | | | | | |
| Phase 3 | | | | | |

---

## Open Findings

{All unresolved findings in severity order — Critical first, then High, Medium, Low.
Use the format defined in finding-format.md.
Suspicious Intent findings are listed first within each severity tier.}

---

## Disproved Findings

~~{Struck-through finding ID and title}~~
Disproved: {what counter-evidence was found and where}

---

## Resolved Findings

~~{Struck-through finding ID and title}~~
Resolved: {what was fixed and when}

---

## Unresolved Uncertain

{Findings that have real evidence but exploitability could not be determined.
These are not resolved. They are explicitly flagged for follow-up.}

**[VULN-{pass}-{n}] {Title}**
Why unresolved: {what context would be needed to classify this}

---

## Product Owner Q&A Log

{Every question asked and every answer received, in order.
This is a permanent record. Entries are never deleted.}

**Q{n} — Pass {pass}:** {question}
**A:** {user's answer}

---

## Scope Map

{Produced during Phase 0. Not modified after Phase 1 begins.}

### Primary Surfaces
{list}

### Secondary Surfaces
{list}

### Out of Scope
{anything explicitly excluded by the user during Phase 0}
