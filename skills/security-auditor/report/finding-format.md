# Finding Format

Every finding recorded in the report must use this exact format.
A finding that cannot satisfy the evidence gate fields is discarded — not recorded.

## Evidence Gate
Before recording a finding, confirm all three are present:
1. Function-level citation — exact file path and function name
2. Exploit path — how it is actually reachable from an external entry point
3. Confidence classification — with explicit reasoning

If any of the three cannot be filled with real evidence, discard the finding.

## Format

```
**[VULN-{pass}-{n}] {Title}**
Category: Injection / Auth / Secrets / Logic / Intent / Infra / Other
Severity: Critical / High / Medium / Low
Confidence: Confirmed / Likely / Uncertain
Suspicious Intent: Yes / No

Location: `path/to/file` → `function_name()`
Exploit Path: [How an attacker reaches and triggers this from an external entry point.
Specific. Not theoretical. If you cannot describe the path, the finding is discarded.]
Evidence: [What in the code supports this finding. Describe precisely.]
Impact: [What breaks, leaks, or gets bypassed if exploited]
Open Questions: [What you need from the user to fully classify this. Leave blank if none.]

Status: Open / Disproved / Resolved
```

## Confidence Definitions
- **Confirmed** — exploit path is clear, evidence is unambiguous, no assumptions required
- **Likely** — exploit path is probable, evidence is strong, one reasonable assumption required
- **Uncertain** — evidence exists but exploitability depends on context you do not have

## Pass Numbering
- Phase 1 findings: VULN-1-1, VULN-1-2, VULN-1-3 ...
- Phase 2 findings: VULN-2-1, VULN-2-2 ...
- Phase 3 findings: VULN-3-1, VULN-3-2 ...

Never renumber findings across passes. IDs are permanent.
