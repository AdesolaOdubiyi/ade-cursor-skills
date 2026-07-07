# Build with Confidence — Examples

## Example 1: Simple API Endpoint (30 min)

### Feature Request
```
/build-with-confidence Add a POST /api/users/:id/avatar endpoint that accepts 
an image file, validates it (size, format), stores it, and returns the URL.
```

### Phase 1: Context (5 min)

Claude asks:
- What image formats? (JPG, PNG, WebP)
- Max file size? (5MB)
- Storage backend? (S3, disk, DB)
- Existing auth? (Bearer token?)
- Async or sync upload?

**CONTEXT.md** (documented):
```markdown
# POST /api/users/:id/avatar

**Feature**: Upload user avatar image

**Scope**: Auth service, storage service, image validation library

**Constraints**:
- Max 5MB, formats: JPEG, PNG, WebP
- Must validate file magic bytes (not just extension)
- Sync upload (not queued)
- Requires Bearer token auth

**Success Criteria**:
1. Endpoint returns 200 + avatar URL on success
2. Returns 413 on oversized file
3. Returns 422 on invalid format
4. Returns 401 if unauthenticated
5. Old avatar deleted when new one uploaded
```

### Phase 1b: Research (parallel)

Claude searches for:
- Image validation best practices (magic bytes, OWASP)
- S3 upload patterns (presigned URLs vs server-side)
- Common pitfalls (EXIF data leaks, zip-bomb attacks)

**RESEARCH.md** summary:
```markdown
- Use magic byte validation, not extension checking
- Reject EXIF data to prevent metadata leaks
- Set strict Content-Type validation (whitelist)
- Implement storage path randomization
- Reference: OWASP File Upload Cheat Sheet
```

### Phase 2: TDD Slices (15 min)

```
Slice 1 (Happy path): Valid JPEG, under size limit
  Test: POST /api/users/123/avatar with valid JPEG file
  Expect: 200, returns avatar URL
  Implementation: Upload to S3, save URL to DB

Slice 2 (Invalid format): POST with ZIP file
  Test: POST with ZIP masquerading as JPEG
  Expect: 422, "Invalid image format"
  Implementation: Add magic byte validation

Slice 3 (Oversized): 10MB JPEG
  Test: POST with file > 5MB
  Expect: 413, "File too large"
  Implementation: Check file size before upload

Slice 4 (No auth): Missing Bearer token
  Test: POST without Authorization header
  Expect: 401, "Unauthorized"
  Implementation: Auth middleware already in place, verify it's called

Slice 5 (Update): Upload new avatar for user with existing avatar
  Test: POST to same user twice
  Expect: 200, old image deleted, new image returned
  Implementation: Query old avatar, delete from S3, upload new one
```

All tests green ✓

### Phase 3: Review Loop (5 min)

**Backend Pre-Merge**:
- Findings: "Missing error handling for S3 timeout"
- Fix: Add 30-second timeout on S3 upload call

**Security Auditor**:
- Findings: "EXIF data not stripped from uploaded images"
- Fix: Add image re-encoding step (removes EXIF)

Re-run reviews → Zero findings ✓

### Phase 4: Acceptance Testing (3 min)

E2E test:
```
1. User logs in (auth token)
2. Upload avatar image via web UI
3. Verify avatar appears in user profile
4. Update avatar with different image
5. Verify old image removed
```

Test passes ✓

### Phase 5: Ship (2 min)

Commit:
```
git commit -m "Add user avatar upload endpoint

- POST /api/users/:id/avatar accepts image upload
- Validates format (JPG, PNG, WebP) and size (max 5MB)
- Strips EXIF data before storage
- Deletes old avatar on update
- All tests passing (5 TDD slices + E2E)"
```

---

## Example 2: Complex Business Logic (2 hours)

### Feature Request
```
/build-with-confidence Implement a subscription renewal system that handles:
- Auto-renewal every month
- Proration on upgrades/downgrades
- Failed payment retry logic
- Cancellation with refund windows
- Email notifications at each step
```

### Phase 1: Context (15 min)

Claude asks:
- Billing provider? (Stripe, custom)
- Proration model? (daily, monthly)
- Refund window? (7, 30 days?)
- Retry strategy? (immediately, after 3 days?)
- Email templates?

**CONTEXT.md**: Captures subscription state machine, refund policy, retry policy, integrations.

### Phase 1b: Research (parallel)

Claude searches for:
- Subscription renewal patterns (SaaS)
- Proration formulas (daily pro-rata)
- Idempotency in payment retries
- Failed payment recovery rates (industry benchmarks)

### Phase 2: TDD Slices (60 min)

```
Slice 1 (Happy path): Auto-renew on schedule
  Test: Subscription due → trigger renewal → charge succeeds
  
Slice 2 (Empty state): Brand new subscription
  Test: Create subscription → no renewal yet → first cycle not charged
  
Slice 3 (Upgrade): Mid-month plan upgrade
  Test: Upgrade from $10/mo to $20/mo → calculate prorated charge
  Expect: ($20 - $10) * remaining_days/month
  
Slice 4 (Downgrade): Plan downgrade with credit
  Test: Downgrade from $20/mo to $10/mo → credit account
  Expect: Credit issued if overpaid
  
Slice 5 (Payment failed): Charge declined
  Test: renewal_charge() fails with Stripe error
  Expect: Retry scheduled, email sent to user
  
Slice 6 (Retry success): Retry after initial failure
  Test: Failed charge → user fixes payment method → retry succeeds
  Expect: Subscription renewed, success email sent
  
Slice 7 (Exceeded retries): Multiple failures
  Test: 3 failed retries over 30 days
  Expect: Subscription canceled, cancellation email sent
  
Slice 8 (Cancellation): User-initiated cancellation
  Test: User cancels → within refund window
  Expect: Refund issued, account deleted at period end
  
Slice 9 (Cancellation past window): Cancellation outside refund period
  Test: User cancels → outside 7-day window
  Expect: No refund, account deleted at period end
  
Slice 10 (Duplicate renewal): Idempotency
  Test: Renewal triggered twice (network retry)
  Expect: Charged once (idempotency key prevents double-charge)
```

All tests green ✓

### Phase 3: Review Loop (20 min)

**Backend Pre-Merge** (Iteration 1):
- "Renewal function is 200 lines, split orchestration from billing logic"
- "No logging on Stripe API calls"
- "Race condition: user cancels while renewal in flight"

Fixes applied.

Re-run → More findings:
- "Missing database transaction for subscription state update"

Fix applied.

Re-run → Zero findings ✓

**Security Auditor** (Iteration 1):
- "Payment amounts not validated (could be negative)"
- "No rate limiting on refund requests"

Fixes applied.

Re-run → Zero findings ✓

### Phase 4: Acceptance Testing (10 min)

E2E scenarios:
1. Happy path: Create subscription → auto-renew → success
2. Failed charge recovery: Charge fails → user fixes → renewal succeeds
3. Upgrade flow: Subscribe → upgrade mid-month → charge prorated
4. Refund window: Cancel within 7 days → refund issued
5. No refund: Cancel after 7 days → no refund
6. Cancellation during renewal: User cancels while renewal in flight

All pass ✓

### Phase 5: Ship (5 min)

Commit documents state machine, retry policy, and test coverage.

---

## Example 3: Security-Critical Feature (90 min)

### Feature Request
```
/build-with-confidence Implement two-factor authentication (2FA) using TOTP 
(Time-based One-Time Passwords) with backup codes and device trust.
```

### Phase 1: Context (10 min)

Questions:
- QR code generation library?
- Backup codes count? (8, 16?)
- Device trust duration? (30 days?)
- Allow password reset without 2FA? (no, security policy)

**CONTEXT.md**: 2FA policy, security constraints, threat model.

### Phase 1b: Research (parallel)

Claude searches:
- TOTP RFC 6238 (timing window, skew tolerance)
- NIST password guidelines on 2FA
- OWASP 2FA best practices
- Time sync issues (clock skew handling)
- Backup code generation (entropy, format)

### Phase 2: TDD Slices (40 min)

```
Slice 1 (Happy path): Enable 2FA, verify with TOTP
Slice 2 (Empty state): User without 2FA enabled
Slice 3 (Invalid code): Wrong TOTP code
Slice 4 (Expired code): Old TOTP (>30 seconds) 
Slice 5 (Time skew): Device clock 60 seconds off
Slice 6 (Backup code): Use backup code to verify
Slice 7 (Backup exhaustion): All backup codes used
Slice 8 (Reset 2FA): Disable and re-enable
Slice 9 (Device trust): Mark device as trusted, skip 2FA on next login
Slice 10 (Concurrent login): Same user logging in from 2 devices simultaneously
```

All tests green ✓

### Phase 3: Review Loop (25 min)

**Backend Pre-Merge**:
- "TOTP secret not encrypted in DB"
- "No rate limiting on verification attempts (brute force vulnerability)"

Fixes applied, re-run → zero findings ✓

**Security Auditor**:
- "Backup codes stored in plaintext"
- "Device trust token not scoped to device fingerprint"
- "No log on 2FA enable/disable events"

Fixes applied, re-run → zero findings ✓

### Phase 4: Acceptance Testing (10 min)

E2E:
1. User enables 2FA via authenticator app
2. User logs in, prompted for TOTP code
3. User enters backup code if TOTP unavailable
4. User marks device as trusted, skips 2FA on next login

All pass ✓

### Phase 5: Ship (5 min)

---

## How to Start

Pick your feature complexity:

| Complexity | Time | Example |
|---|---|---|
| Simple | 30 min | Single endpoint, no external APIs, straightforward validation |
| Medium | 1–2 hours | Feature with state machine, integrations, or complex logic |
| Complex | 2–4+ hours | Security-critical, distributed, high-stakes logic |

Then invoke:

```
/build-with-confidence <your feature description>
```

Claude will guide you through all 5 phases, asking questions as needed and keeping you in the loop.

---

## Tips

1. **Start with a clear description** — "Add password reset" is better than "Fix auth stuff"
2. **Be specific about constraints** — "Max 5MB, JPEG only" saves back-and-forth
3. **Mention integrations early** — "Uses Stripe for billing" — so research is targeted
4. **Review findings carefully** — If a finding repeats after your fix, ask why before dismissing it
5. **Use acceptance tests as confidence** — If E2E passes, the feature works

---

## See Also

- **Simple feature?** Skip `/build-with-confidence`, use `/tdd` directly
- **Need security deep-dive?** Pair with `/security-auditor` manually
- **Just refactoring?** Use `/design-to-build` for architecture, then `/build-with-confidence` for implementation
