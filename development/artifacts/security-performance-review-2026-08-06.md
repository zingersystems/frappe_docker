# Custom Apps Security and Performance Review

Date: 2026-08-06

Scope:

- `academia_core`
- `academia_catuc`
- `frappe_pay_connect`

Upstream `frappe`, `erpnext`, `education`, and `payments` are dependencies and are outside the edit scope.

## Findings

### Medium: Mutating guest APIs accepted unsafe HTTP methods

Status: Resolved in the first implementation tranche.

Frappe whitelist decorators default to `GET`, `POST`, `PUT`, and `DELETE`. Application and payment methods performed mutations while relying on body-level POST checks or having no explicit method restriction at dispatch.

Affected surfaces included:

- Application draft creation and access verification
- Application session extension and sign-out
- Wizard saves, uploads, removal, reopening, and submission
- Access-code recovery and protected document actions
- Staff access recovery and offline-payment verification
- Payment initiation, authoritative refresh, and retry

Remediation:

- Restricted every classified mutation and POST-only protected document action with `methods=["POST"]` at the Frappe whitelist boundary.
- Kept genuine reads such as form discovery, application load/status, validation, payment options, and payment status GET-compatible.
- Retained existing inner request checks where they provide defense in depth.

Evidence:

- The new metadata regressions initially reported `['GET', 'POST', 'PUT', 'DELETE']` for every tested mutation.
- After remediation, Core access tests passed 18/18 and Pay Connect lifecycle tests passed 11/11.

### Medium: Upload limits were enforced only after Base64 decoding

Status: Resolved in the first implementation tranche.

Core wizard attachments and Pay Connect payment proofs decoded the complete submitted Base64 body before checking the 5 MB decoded-file limit. A large request could therefore force unnecessary decoded-memory allocation before rejection.

Remediation:

- Calculate the maximum possible encoded length from the configured decoded-file limit.
- Reject encoded bodies exceeding that bound before calling `base64.b64decode`.
- Use strict Base64 validation and return controlled validation errors for malformed content.
- Retain the decoded-size check to cover padding and boundary cases.

Evidence:

- Tests assert that `base64.b64decode` is not called for an impossible-to-fit encoded payload.
- Core upload security tests passed 2/2.
- Pay Connect lifecycle tests passed 11/11.

### Medium: Public access recovery had no abuse throttle

Status: Resolved in the second implementation tranche.

The guest access-code recovery endpoint could repeatedly rotate a matching applicant's credential, revoke active
sessions, and queue email. Its neutral response prevented identifier disclosure but did not constrain targeted
mailbox or credential churn, identifier spraying, or repeated database work.

Remediation:

- Limited each normalized Application number to three recovery requests per hour across source IPs.
- Limited each source IP to ten recovery requests per hour to constrain identifier spraying.
- Normalized casing and surrounding whitespace before selecting the per-application bucket.
- Applied rejection before applicant lookup, credential rotation, email dispatch, or commit.

Evidence:

- A focused regression submits lowercase, whitespace-padded, and canonical forms of one Application number and
	confirms the fourth request raises `RateLimitExceededError`.
- The same regression confirms only the first three requests reach applicant lookup.
- Core access tests passed 19/19.

## Verification

Completed:

- `academia_core.admissions.test_access`: 18 passed.
- `academia_core.admissions.test_upload_security`: 2 passed.
- `academia_core.admissions.test_wizard`: 53 passed.
- `academia_core.admissions.test_payment_renderer`: 4 passed.
- `frappe_pay_connect.payments.test_lifecycle`: 11 passed.
- Python compilation for `academia_core` and `frappe_pay_connect`: passed.
- Editor diagnostics for all touched Python files: clean.

## Pending Review Work

- Rate limiting beyond public access recovery
- Callback signature/configuration and browser-return trust boundaries
- Payment and proof concurrency/idempotency races
- File signature/MIME validation and orphan cleanup on failed persistence
- Secret and sensitive-log redaction audit
- Cross-applicant and cross-policy authorization negatives
- Portal query-count baseline and N+1 remediation
- Wizard request-local query/state reuse
- Payment discovery and reconciliation profiling
- Three-pathway HND, Undergraduate, and Postgraduate browser regression matrix
