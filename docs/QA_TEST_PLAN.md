# DocFlow — QA Test Plan

## 1. Purpose

This plan defines functional, authorization, security, integration, reliability, and end-to-end checks for DocFlow. It is a baseline test plan; test cases should be updated as API contracts and business rules are finalized.

## 2. Test scope

### In scope

- Tenant provisioning and tenant isolation.
- Authentication, invitations, sessions, and refresh-token rotation.
- Role and permission management.
- Folder and document operations.
- Uploads, downloads, and version history.
- Document submission, approval, rejection, archive, and restore.
- Audit events and notifications.
- Outbox publishing and message consumption.
- Rate limiting, caching, observability, and error handling.

### Out of scope until implemented

- Production-scale load certification.
- External identity providers not selected for the release.
- Compliance certification and formal penetration testing.
- Retention/legal-hold behavior until policy is approved.

## 3. Test environment

Recommended local dependencies:

- .NET 10 API/worker processes.
- SQL Server Developer.
- RabbitMQ and Management UI.
- MinIO.
- Redis.
- Mailpit.
- Docker Compose.
- xUnit and Testcontainers.

Use isolated test tenants and generated test data. Never use real customer documents or production credentials in local test runs.

## 4. Test personas

| Persona | Expected access |
|---|---|
| Tenant Admin | Tenant administration and broad tenant-level access |
| Document Manager | Folder/document lifecycle within assigned scope |
| Editor | Create, edit permitted metadata, upload, submit |
| Reviewer | Review, approve, reject eligible submissions |
| Viewer | Read-only permitted document access |
| Auditor | Permitted audit events and reports |

Create at least two tenants, Tenant A and Tenant B, with users assigned to each. Cross-tenant negative tests are mandatory.

## 5. Core test cases

### Authentication and invitations

| ID | Scenario | Expected result |
|---|---|---|
| AUTH-001 | Sign in with valid active user | Authentication succeeds |
| AUTH-002 | Sign in with incorrect password | Generic authentication failure; no token issued |
| AUTH-003 | Sign in with deactivated user | Authentication denied |
| AUTH-004 | Accept valid invitation | User is activated and invitation token is consumed |
| AUTH-005 | Reuse accepted invitation token | Request rejected |
| AUTH-006 | Accept expired invitation | Request rejected |
| AUTH-007 | Refresh valid refresh token | New token issued and old token rotated/revoked |
| AUTH-008 | Reuse rotated refresh token | Reuse policy is triggered; affected session family handled |
| AUTH-009 | Logout | Refresh session is revoked |
| AUTH-010 | Use expired access token | Protected endpoint rejects request |

### Tenant isolation

| ID | Scenario | Expected result |
|---|---|---|
| TEN-001 | Tenant A user reads Tenant A document | Allowed when permission exists |
| TEN-002 | Tenant A user reads Tenant B document by ID | Denied without leaking resource data |
| TEN-003 | Tenant A user updates Tenant B document | Denied |
| TEN-004 | Tenant A user downloads Tenant B file | Denied |
| TEN-005 | Client changes tenant ID in request body | Server does not trust the supplied tenant ID |
| TEN-006 | Tenant A user queries Tenant B audit events | Denied |

### RBAC and authorization

| ID | Scenario | Expected result |
|---|---|---|
| RBAC-001 | Viewer attempts document create | Forbidden |
| RBAC-002 | Viewer attempts document upload | Forbidden |
| RBAC-003 | Editor creates permitted draft | Allowed |
| RBAC-004 | Editor attempts approval | Forbidden unless explicitly granted by policy; default is denied |
| RBAC-005 | Reviewer reviews eligible submission | Allowed |
| RBAC-006 | Reviewer attempts to approve own submission | Denied |
| RBAC-007 | Auditor queries permitted audit events | Allowed |
| RBAC-008 | Auditor downloads document without file permission | Denied |
| RBAC-009 | User loses permission while cache is populated | Access is denied within the documented invalidation/expiry guarantee |
| RBAC-010 | User attempts an action by calling API directly while UI hides it | API denies unauthorized action |

### Folders and documents

| ID | Scenario | Expected result |
|---|---|---|
| DOC-001 | Create folder with valid parent | Folder created in correct tenant |
| DOC-002 | Create folder under another tenant's parent | Denied |
| DOC-003 | Create document with valid metadata | Draft created |
| DOC-004 | Update permitted draft metadata | Changes saved |
| DOC-005 | Update approved document using generic status DTO | Invalid transition rejected |
| DOC-006 | Search documents | Results respect tenant and resource scope |
| DOC-007 | Pagination with stable sort | No unexpected duplicates or skipped rows under stable data |

### Uploads and versions

| ID | Scenario | Expected result |
|---|---|---|
| FILE-001 | Upload permitted valid file | Object and version metadata created |
| FILE-002 | Upload file exceeding configured size | Rejected with clear validation response |
| FILE-003 | Upload disallowed extension | Rejected |
| FILE-004 | MIME type does not match file signature | Rejected or quarantined according to policy |
| FILE-005 | Filename contains path traversal characters | Stored using generated safe object key |
| FILE-006 | SQL metadata save fails after object upload | Orphan cleanup/reconciliation path is triggered |
| FILE-007 | Upload new version | Previous binary remains unchanged |
| FILE-008 | Unauthorized user downloads file | Denied before file content is returned |
| FILE-009 | Client disconnects during upload | Cancellation and temporary-object cleanup are handled |

### Approval workflow

| ID | Scenario | Expected result |
|---|---|---|
| WF-001 | Editor submits valid draft | State changes to Submitted |
| WF-002 | Submit draft without required version/metadata | Rejected with validation details |
| WF-003 | Reviewer approves submitted version | State changes to Approved |
| WF-004 | Reviewer rejects without reason | Rejected by validation |
| WF-005 | Reviewer rejects with reason | State changes to Rejected and reason is recorded |
| WF-006 | Editor revises rejected document | Document returns to Draft according to workflow |
| WF-007 | Approve a version after a newer version is submitted | Stale decision is rejected or handled by explicit concurrency policy |
| WF-008 | Archive approved document | State changes to Archived |
| WF-009 | Restore archived document | State changes to Draft or the explicitly defined restore state |

### Audit and notifications

| ID | Scenario | Expected result |
|---|---|---|
| AUD-001 | User creates document | Audit event records actor, tenant, action, resource, and time |
| AUD-002 | Approval/rejection occurs | Decision and outcome are audited |
| AUD-003 | Unauthorized operation is attempted | Security/audit event recorded according to policy |
| AUD-004 | Audit query by unauthorized user | Denied |
| AUD-005 | Logs are inspected | No passwords, tokens, or secrets are present |
| NOT-001 | Document submitted | Intended reviewer notification is generated |
| NOT-002 | Document approved/rejected | Intended editor notification is generated |
| NOT-003 | Notification is marked read | Read state is persisted |

### Outbox and messaging

| ID | Scenario | Expected result |
|---|---|---|
| MSG-001 | Business transaction commits with outbox row | Both are committed |
| MSG-002 | Business transaction rolls back | Outbox row is not committed |
| MSG-003 | Publisher sends message successfully | Row is marked processed after broker confirmation |
| MSG-004 | Publisher crashes after publish before marking row | Duplicate may occur; consumer remains idempotent |
| MSG-005 | RabbitMQ unavailable | Outbox remains pending and retries are bounded |
| MSG-006 | Consumer receives same message twice | Business side effect occurs once |
| MSG-007 | Poison message repeatedly fails | Message is moved to configured dead-letter handling |

### Rate limiting, caching, and resilience

| ID | Scenario | Expected result |
|---|---|---|
| REL-001 | Client exceeds configured rate limit | HTTP 429 and appropriate retry information |
| REL-002 | One tenant exceeds its quota | Other tenants are not unintentionally throttled by tenant-specific policy |
| REL-003 | Permission changes | Permission cache is invalidated or expires within documented bound |
| REL-004 | Redis is unavailable | Application follows documented fallback/failure policy |
| REL-005 | Downstream dependency times out | Request fails within configured timeout |
| REL-006 | Retryable transient failure occurs | Bounded retry/backoff is applied |
| REL-007 | Cancellation token is triggered | Work is cancelled where safe |

## 6. End-to-end scenarios

### E2E-001: Invitation to first login

1. Tenant Admin invites a new user.
2. User accepts a valid invitation.
3. User signs in.
4. User sees navigation appropriate to assigned role.
5. User cannot access another tenant's resources.

### E2E-002: Document approval

1. Editor creates a draft.
2. Editor uploads version 1.
3. Editor submits the document.
4. Reviewer opens the submitted version.
5. Reviewer rejects with a reason.
6. Editor revises and uploads version 2.
7. Editor resubmits.
8. Reviewer approves version 2.
9. Audit history records both decisions and version references.
10. Notifications are generated for the appropriate users.

### E2E-003: Outbox recovery

1. Perform a business operation that writes an outbox event.
2. Stop RabbitMQ before the publisher can deliver.
3. Verify the outbox row remains pending.
4. Restart RabbitMQ and publisher.
5. Verify eventual publication.
6. Replay the message and verify consumer idempotency.

### E2E-004: Cross-tenant security

1. Create Tenant A and Tenant B.
2. Create a document in Tenant B.
3. Authenticate as a Tenant A user.
4. Attempt to read, update, download, and audit the Tenant B document.
5. Verify all unauthorized operations are denied and no sensitive metadata leaks.

## 7. Security test checklist

- [ ] Broken object-level authorization / IDOR.
- [ ] Cross-tenant reads and writes.
- [ ] Privilege escalation through role assignment.
- [ ] Direct API invocation bypassing UI visibility.
- [ ] CSRF for cookie-authenticated state-changing requests.
- [ ] XSS-safe rendering of document metadata/comments.
- [ ] SQL injection resistance through parameterized queries.
- [ ] Upload extension/MIME/signature validation.
- [ ] Path traversal and unsafe filename handling.
- [ ] Refresh-token rotation and reuse detection.
- [ ] Invitation token expiry and one-time use.
- [ ] Deactivated-user session revocation.
- [ ] Sensitive data excluded from logs and error responses.
- [ ] Rate limiting on authentication and expensive endpoints.

## 8. Performance and reliability checks

- Measure document-list query latency with realistic tenant data volume.
- Verify indexes support common tenant-scoped filters and sort orders.
- Test concurrent updates and optimistic concurrency behavior.
- Test large uploads within configured size limits.
- Measure outbox backlog recovery after broker downtime.
- Verify worker retry behavior does not create unbounded message storms.
- Confirm health checks distinguish application readiness from dependency health.

Define measurable thresholds before release; do not invent pass criteria after observing results.

## 9. UI checks

- Navigation matches role permissions.
- Unauthorized actions are hidden or disabled where appropriate.
- API authorization remains effective even if the UI is manipulated.
- Empty, loading, error, and success states are understandable.
- Long filenames and metadata do not break layouts.
- Confirmation is shown for destructive or irreversible actions.
- Rejection reason is visible to the submitting user.
- Version identity is clear during review.

## 10. Entry and exit criteria

### Entry

- Requirements and permission matrix reviewed.
- Test environment available.
- Migrations and seed data applied.
- Test accounts for all six roles created.
- At least two tenants provisioned.

### Exit

- All critical and high-priority tests pass.
- No unresolved critical security defects.
- Cross-tenant isolation tests pass.
- Workflow and versioning tests pass.
- Outbox duplicate/retry tests pass.
- Known limitations and accepted risks documented.

## 11. Defect report template

```text
Title:
Environment/build:
Tenant and role:
Preconditions:
Steps to reproduce:
Expected result:
Actual result:
Frequency:
Severity/priority:
Correlation ID:
Evidence (redacted):
```

Never attach passwords, access tokens, refresh tokens, or real sensitive documents to a defect report.

## 12. Release checklist

- [ ] Configuration and secrets are supplied securely.
- [ ] Database migrations reviewed and tested.
- [ ] Tenant isolation suite passes.
- [ ] Role/permission suite passes.
- [ ] Upload validation and download authorization pass.
- [ ] Workflow transition and concurrency tests pass.
- [ ] Outbox and consumer idempotency tests pass.
- [ ] Audit events are complete and safe.
- [ ] Logs, metrics, traces, and health checks are available.
- [ ] Backup, retention, and recovery expectations are documented.
- [ ] Known issues and operational runbooks are reviewed.
