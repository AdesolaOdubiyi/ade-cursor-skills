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

---

## How to Run a Session

Follow these phases in order. Do not skip phases. Do not merge phases.

### Phase 0 — Attack Surface Census

Before any code review, map what an attacker sees. Output:

```
ATTACK SURFACE MAP
══════════════════
CODE SURFACE
  Public endpoints:      N (unauthenticated)
  Authenticated:         N (require login)
  Admin-only:            N (require elevated privileges)
  File upload points:    N
  External integrations: N
  Background jobs:       N
  WebSocket channels:    N

INFRASTRUCTURE SURFACE
  CI/CD workflows:       N
  Container configs:     N
  IaC configs:           N
  Env/secret files:      N
  Secret management:     [env vars | KMS | vault | unknown]
```

Also detect the tech stack and build an architectural mental model:
- Read README, key config files, entrypoint files
- Map trust boundaries and data flow: where does user input enter? Where does it exit?
- Identify invariants the code relies on

Do NOT begin Phase 1 until the scope map is complete.

---

### Phase 1 — Infrastructure-First Audit

Audit in this order. The real attack surface is often NOT the application code.

**Secrets Archaeology** — Search git history and current files for credentials, API keys, tokens. Patterns: `SECRET`, `PASSWORD`, `API_KEY`, `-----BEGIN`, AWS/GCP/Azure key formats. Check `.env*` files, CI config files, commit history.

**Supply Chain** — Review direct dependencies for: known CVEs (check advisories), `postinstall`/`preinstall` scripts in package manifests, version pins vs floating ranges. Verify lockfile exists and is tracked in git.

**CI/CD Pipeline** — Review workflow files for: `pull_request_target` that checks out PR code (RCE risk), unpinned third-party actions (supply chain risk), secrets exposed via `echo` or `env:` blocks, script injection via `${{ github.event.* }}`.

**Infrastructure Configs** — Docker, Kubernetes, Terraform: containers running as root in production, open network policies, hardcoded secrets in IaC, overly broad IAM permissions.

**Application Code** — OWASP Top 10 pass across all Primary surfaces identified in Phase 0. Injection (SQL, command, SSRF), broken auth, sensitive data exposure, security misconfig, XSS, insecure deserialization.

---

### Phase 2 — Self-Challenge Pass

Run both objectives simultaneously:
1. Attempt to disprove every Uncertain finding from Phase 1 by finding counter-evidence. If disproved, mark Disproved with a note — do not delete.
2. Approach the codebase from new entry points to find what Phase 1 missed.

After Phase 2, apply stopping rules: if all findings are resolved or confirmed, close. If material Uncertain findings remain, proceed to Phase 3.

Wait for the user before proceeding to Phase 3.

---

### Phase 3 — Final Pass (Hard Ceiling)

Same process as Phase 2. After Phase 3, stop regardless. Mark all remaining Uncertain findings as explicitly unresolved. Produce the final report. Do not offer another pass.

---

## Confidence Threshold

A finding must have confidence ≥ 8/10 before it is reported in the main findings. Every finding must include a concrete exploit scenario — specific inputs, attacker position, and what breaks. Plausible-but-vague concerns go in a separate "Low confidence / worth watching" section, not in the main findings.

---

## Variant Analysis

When a finding is VERIFIED, search the entire codebase for the same vulnerability pattern. One confirmed SSRF means there may be 5 more. For each verified finding:
1. Extract the core vulnerability pattern
2. Search for the same pattern across all relevant files
3. Report variants as separate findings linked to the original: "Variant of Finding #N"

---

## Incident Response Playbooks

When a leaked secret is found, include this playbook:
1. **Revoke** the credential immediately (do not wait for the report to be read)
2. **Rotate** — generate a new credential
3. **Scrub history** — `git filter-repo` or BFG Repo-Cleaner to remove the commit
4. **Force-push** the cleaned history
5. **Audit exposure window** — when committed? When removed? Was repo public?
6. **Check for abuse** — review the provider's audit logs for the window

---

## Trend Tracking

If prior reports exist in `.security-reports/`:

```
SECURITY POSTURE TREND
══════════════════════
Compared to last audit ({date}):
  Resolved:    N findings fixed
  Persistent:  N findings still open
  New:         N findings discovered this audit
  Trend:       ↑ IMPROVING / ↓ DEGRADING / → STABLE
```

Save each report to `.security-reports/{date}-{HHMMSS}.json`. If `.security-reports/` is not in `.gitignore`, note it — security reports should stay local.

---

## Core Rules
- Never report a finding you cannot cite at function level
- Never assume product behavior — ask the user
- Never escalate severity without evidence
- Suspicious intent escalates severity by one tier automatically
- The report is cumulative and versioned — never overwrite previous pass findings
- Resolved findings are struck through, not deleted
- Zero noise is more important than zero misses — 3 real findings beats 3 real + 12 theoretical

## Disclaimer
This skill is not a substitute for a professional security audit. It catches common vulnerability patterns but is not comprehensive. For production systems handling sensitive data, payments, or PII, engage a qualified security firm. Use this as a first pass to catch low-hanging fruit between professional audits — not as your only line of defense.
