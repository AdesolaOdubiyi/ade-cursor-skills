# Suspicious Intent Patterns

Apply this checklist throughout all passes on all surfaces.
A match here does not automatically mean malicious — but it must be flagged
separately and the user must be asked about it before it is classified.

Flag a finding as Suspicious Intent when any of the following are present:

## Bypass Patterns
- Auth check is present but a specific parameter, header, or value disables it
- Validation runs but returns success on the failure path
- A condition that should block execution instead continues silently
- A comparison uses a type-unsafe operator where the safer one would work equally well
- An allowlist or denylist contains an entry that seems out of place

## Hardcoded Values in Sensitive Paths
- A hardcoded string is compared against user input in an auth or permission check
- A hardcoded value triggers different behavior than all other inputs
- A commented-out bypass or debug credential exists anywhere in the codebase
- A value described in a comment as temporary or to-be-removed is still active

## Obfuscation and Complexity
- Security-sensitive logic is significantly more complex than the surrounding code
  with no clear reason
- An operation is split across multiple files or indirections in a way that makes
  it hard to trace
- Variable or function names in security-sensitive paths are misleading relative
  to what they actually do
- Dead code exists in a security-sensitive path

## Second-Order and Interaction Effects
- A finding in one module creates exploitable conditions in another module that
  would appear safe in isolation
- A race condition exists that is only exploitable in combination with another
  finding

## Severity Escalation Rule
Any finding flagged as Suspicious Intent is automatically escalated one severity tier:
- Low → Medium
- Medium → High
- High → Critical
- Critical → Critical (stays, but noted as suspected intentional)

Record the escalation in the finding with the note: "Escalated: suspicious intent flag"
