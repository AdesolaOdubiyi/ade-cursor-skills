# Application Code Checklist

Apply this checklist to all Primary surfaces identified in the scope map.
For each item, actively search for violations — do not assume compliance.
Every violation requires function-level citation before it is recorded.

## Authentication and Authorization
- [ ] Auth checks are present on every protected route and cannot be bypassed
      by parameter manipulation, header injection, or request modification
- [ ] Token validation checks signature, expiry, and claims — not just presence
- [ ] Permission checks happen server-side — never trust client-supplied role or
      permission values
- [ ] Session invalidation works on logout and token revoke — not just client-side
- [ ] Password handling uses a modern adaptive hashing algorithm
- [ ] Auth logic contains no hardcoded bypass conditions or master credentials
- [ ] Multi-tenant isolation: one user cannot access another user's data by
      manipulating identifiers in requests

## Injection Surfaces
- [ ] All database queries use parameterized statements or a safe ORM — no
      string concatenation with user input
- [ ] All shell command execution sanitizes input — no user data in exec calls
- [ ] All template rendering is safe from server-side template injection
- [ ] All XML/YAML/JSON parsing is safe from entity expansion and deserialization
      attacks
- [ ] All redirect targets are validated against an allowlist

## Input Validation
- [ ] All external input is validated before use — type, length, format, range
- [ ] Validation happens server-side — client-side validation is not relied upon
- [ ] File uploads validate type by content, not extension — file is scanned or
      sandboxed before processing
- [ ] All input validation failures return a safe error — no stack traces or
      internal paths in responses

## Secrets and Credentials
- [ ] No secrets, API keys, tokens, or credentials are hardcoded in source
- [ ] No secrets are logged — not in debug, info, or error logs
- [ ] No secrets are returned in API responses
- [ ] Environment variables that should be server-only are never exposed to the client
- [ ] Dependency manifests contain no known vulnerable packages

## Error Handling and Logging
- [ ] No exception handler silently swallows errors in security-sensitive paths
- [ ] Error responses do not expose internal paths, stack traces, or system info
- [ ] All security-relevant events are logged with sufficient context
- [ ] Log entries do not contain sensitive data

## Business Logic
- [ ] Rate limiting or abuse prevention exists on all sensitive operations
- [ ] State transitions cannot be skipped or reversed by replaying requests
- [ ] Numeric operations handling money or quantities are safe from overflow and
      precision errors
- [ ] Any operation that should be atomic is protected from race conditions

## External Service Calls
- [ ] All outbound HTTP calls validate TLS certificates — no verification disabled
- [ ] Webhook receivers validate signatures before processing payloads
- [ ] SSRF protection exists on any endpoint that fetches a user-supplied URL
- [ ] Third-party SDK configuration does not expose credentials or bypass security
      features
