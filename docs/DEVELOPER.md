# DocFlow — Developer Documentation

## 1. Purpose

DocFlow is a multi-tenant document management and approval platform. It is designed as a practical backend project covering ASP.NET Core, SQL Server, Dapper, authorization, file storage, messaging, reliability, testing, and observability.

This document describes the proposed technical baseline. It is not proof that every feature has already been implemented.

## 2. Core capabilities

- Tenant provisioning and tenant-scoped users.
- User invitations, activation/deactivation, authentication, and session management.
- Role-based permissions with resource-level authorization.
- Folder hierarchy and document metadata.
- Document uploads and immutable version history.
- Submission, review, approval, rejection, archive, and restore.
- Audit trail and notifications.
- Reliable asynchronous integration through an outbox.

## 3. Technology choices

| Concern | Choice |
|---|---|
| Runtime/API | .NET 10 / ASP.NET Core |
| Relational database | SQL Server |
| Data access | Dapper |
| Messaging | RabbitMQ |
| Object storage | MinIO |
| Cache | Redis |
| Reverse proxy | YARP |
| Tests | xUnit, Testcontainers |
| Telemetry | Serilog, OpenTelemetry, Prometheus, Grafana |
| Local dependencies | Docker Compose |
| Email sandbox | Mailpit |

RabbitMQ and MinIO are local alternatives for learning patterns used with Azure Service Bus and Azure Blob Storage. Their semantics and operational features are not identical.

## 4. Architecture

Begin with a modular monolith. Keep module boundaries clear and extract services only when deployment, scaling, ownership, or reliability needs justify the added complexity.

### Dependency direction

- API depends on Application.
- Infrastructure implements interfaces defined by Application and depends on Domain.
- Application depends on Domain.
- Domain has no dependency on outer layers.

Avoid adding repository and unit-of-work abstractions automatically. Use abstractions where they protect a meaningful boundary or improve testability.

### Suggested solution layout

```text
DocFlow/
├── src/
│   ├── Gateway/
│   ├── Services/
│   │   ├── Identity/
│   │   │   ├── Identity.API/
│   │   │   ├── Identity.Application/
│   │   │   ├── Identity.Domain/
│   │   │   └── Identity.Infrastructure/
│   │   ├── Document/
│   │   │   ├── Document.API/
│   │   │   ├── Document.Application/
│   │   │   ├── Document.Domain/
│   │   │   └── Document.Infrastructure/
│   │   └── Notification/
│   │       ├── Notification.Worker/
│   │       ├── Notification.Domain/
│   │       └── Notification.Infrastructure/
│   └── Shared/
│       ├── Contracts/
│       └── Observability/
├── tests/
├── infrastructure/
└── docs/
```

## 5. Roles and authorization

### Initial roles

| Role | Responsibilities |
|---|---|
| Tenant Admin | Tenant configuration, users, invitations, roles, permissions, broad administration |
| Document Manager | Folder/document lifecycle, metadata, upload, archive and restore within scope |
| Editor | Create/edit permitted documents, upload versions, submit for approval |
| Reviewer | Approve/reject eligible submitted versions |
| Viewer | Read-only browse, preview, and download where permitted |
| Auditor | Read permitted audit events and reports |

Do not assume a role name alone grants access to every resource. Authorization should evaluate:

1. Is the user authenticated and active?
2. Does the user belong to the tenant that owns the resource?
3. Does the user have the required action permission?
4. Is the resource within the user's allowed scope?
5. Is the requested operation valid for the current document state?

The UI may hide actions the user cannot perform, but the API must enforce the same rules independently.

### Example permission names

- `Dashboard.View`
- `Document.View`
- `Document.Create`
- `Document.Edit`
- `Document.Upload`
- `Document.Download`
- `Document.Submit`
- `Document.Archive`
- `Folder.View`
- `Folder.Create`
- `Folder.Edit`
- `Approval.View`
- `Approval.Approve`
- `Approval.Reject`
- `User.View`
- `User.Create`
- `User.Edit`
- `User.Deactivate`
- `User.AssignRole`
- `Role.View`
- `Role.Edit`
- `Role.AssignPermission`
- `Audit.View`
- `Notification.View`
- `Notification.MarkRead`
- `TenantSettings.View`
- `TenantSettings.Edit`

## 6. Domain rules and document lifecycle

```text
Draft -> Submitted -> Approved -> Archived
                 \-> Rejected -> Draft
Draft --------------------------> Archived
Archived -----------------------> Draft (restore)
```

Rules:

- Do not allow clients to set arbitrary status values through a general update DTO.
- Use explicit commands/domain operations for submit, approve, reject, archive, and restore.
- Approval applies to a specific immutable version.
- A reviewer cannot approve their own submission.
- Rejection requires a meaningful reason.
- Uploading a new version must not overwrite the binary of an earlier version.
- Preserve previous versions according to retention policy.
- Validate state transitions in the domain/application layer, not only in UI code.

## 7. Tenant isolation

- Tenant identity must come from validated claims or another trusted server-side context.
- Never trust a tenant ID supplied by the client as the authority for data access.
- Include tenant scope in SQL predicates and relevant unique constraints.
- Include tenant and user identifiers in cache keys.
- Test cross-tenant access explicitly for reads, writes, downloads, and audit queries.
- Background workers must preserve tenant context from the event payload and validate it before processing.

## 8. Identity and session security

### Invitations

- Generate high-entropy, one-time invitation tokens.
- Store only a cryptographic hash of the token.
- Enforce expiry and revoke the token after successful acceptance.
- Do not reveal whether arbitrary email addresses are registered through overly specific responses.

### Passwords and refresh tokens

- Use a purpose-built password hashing algorithm; never store plaintext passwords.
- Use opaque, high-entropy refresh tokens and store their hashes.
- Rotate refresh tokens on use and revoke the token family when reuse is detected, according to the chosen policy.
- Deactivated users must not log in or refresh sessions.
- Revoke active sessions when a user is deactivated or security-sensitive permissions change.

### Access tokens

Short-lived JWT access tokens may remain valid until expiry after logout unless a revocation strategy is implemented. Document this trade-off and test the selected behavior.

### Cookies and antiforgery

For cookie-based authentication, use `Secure`, `HttpOnly`, and an appropriate `SameSite` policy. Protect state-changing requests with antiforgery validation where required.

## 9. Illustrative API surface

These routes are examples for planning, not finalized contracts.

| Area | Example route | Purpose |
|---|---|---|
| Authentication | `POST /api/auth/login` | Sign in |
| Authentication | `POST /api/auth/refresh` | Rotate refresh token |
| Authentication | `POST /api/auth/logout` | Revoke session |
| Users | `GET /api/users` | List tenant users |
| Invitations | `POST /api/invitations` | Invite a user |
| Invitations | `POST /api/invitations/accept` | Accept invitation |
| Roles | `GET /api/roles` | List roles |
| Permissions | `GET /api/permissions` | List permission catalogue |
| Folders | `POST /api/folders` | Create folder |
| Documents | `POST /api/documents` | Create document metadata |
| Documents | `GET /api/documents/{id}` | Get document details |
| Versions | `POST /api/documents/{id}/versions` | Upload a new version |
| Workflow | `POST /api/documents/{id}/submit` | Submit for review |
| Workflow | `POST /api/documents/{id}/approve` | Approve submitted version |
| Workflow | `POST /api/documents/{id}/reject` | Reject with reason |
| Audit | `GET /api/audit-events` | Query permitted audit events |

Use consistent error responses, validation, pagination, and correlation IDs.

## 10. SQL Server and Dapper

- Use parameterized queries for all user-controlled values.
- Keep SQL readable and review execution plans for important queries.
- Add indexes based on actual filters, joins, and ordering.
- Include tenant keys in relevant indexes and constraints.
- Use explicit transactions for multi-step relational changes.
- Understand isolation levels, deadlocks, optimistic concurrency, and retry boundaries.
- Do not retry non-idempotent writes blindly.
- Use `rowversion` or another concurrency token where concurrent updates are possible.
- Pass cancellation tokens through request and data-access paths.

## 11. File upload and object storage

SQL Server and object storage cannot participate in one ordinary ACID transaction.

Recommended flow:

1. Authenticate and authorize the upload.
2. Validate size, extension, declared MIME type, and file signature.
3. Generate a server-side object key.
4. Stream the file to temporary or final object storage.
5. Persist document/version metadata in SQL.
6. If metadata persistence fails, schedule or perform object cleanup.
7. Run reconciliation to detect orphaned objects and missing binaries.

Never use the original filename as an object-storage path. Apply download authorization before issuing a stream or short-lived access URL. Consider malware scanning for untrusted uploads.

## 12. Outbox and RabbitMQ

The outbox solves the dual-write problem between a database transaction and message publication.

1. Begin a SQL transaction.
2. Persist the business change.
3. Insert an outbox row in the same transaction.
4. Commit the transaction.
5. A background publisher reads pending outbox rows.
6. It publishes messages to RabbitMQ.
7. After broker confirmation, it marks the outbox row as processed.
8. Consumers handle duplicate delivery idempotently.

A crash after publish but before marking the row processed can cause a duplicate. This is expected; consumers must use message IDs or business idempotency keys. Add retry limits, backoff, poison-message handling, and dead-letter queues.

## 13. Caching and rate limiting

- Use tenant-aware cache keys.
- Cache permission data only with a clear invalidation strategy.
- Invalidate permission caches after role/permission changes and user deactivation.
- Consider cache stampede protection for expensive reads.
- Apply rate limits by IP, user, tenant, or endpoint depending on the threat.
- Return `429 Too Many Requests` and a useful `Retry-After` value when appropriate.

## 14. Audit and observability

An audit event should capture, where applicable:

- Tenant ID
- Actor/user ID
- Action
- Resource type and ID
- Timestamp
- Outcome
- Correlation/trace ID
- Safe contextual metadata

Do not put passwords, tokens, secrets, or document contents in logs. Use structured logs, distributed traces, metrics, and health checks. Keep audit records separate from ordinary diagnostic logs where retention and access requirements differ.

## 15. Resilience

Use bounded retries with exponential backoff and jitter where appropriate. Set timeouts. Use circuit breakers and bulkheads for remote dependencies when justified. Propagate cancellation tokens. Design compensation or cleanup for workflows that span SQL and object storage.

## 16. Testing strategy

- **Unit tests:** domain invariants, state transitions, permission decisions.
- **Integration tests:** SQL queries, transactions, Dapper mappings, outbox persistence.
- **API tests:** authentication, authorization, validation, status codes.
- **Security tests:** tenant isolation, IDOR, privilege escalation, upload validation, refresh-token reuse.
- **Messaging tests:** duplicate delivery, retry, dead-letter behavior, idempotent consumers.
- **End-to-end tests:** invitation through approval and audit verification.

Use Testcontainers for dependencies where practical. Do not rely only on mocks for database behavior.

## 17. Decisions to finalize

- Whether email addresses are unique globally or per tenant.
- Whether Editors may create folders.
- Whether Document Managers or Tenant Admins may approve documents by default.
- Whether folders are tenant-wide or have granular sharing.
- Maximum upload size and permitted file types.
- Retention and deletion policy for old versions and audit records.
- Download streaming versus short-lived signed URLs.
- Access-token revocation strategy.
- Whether antivirus scanning is mandatory.

## 18. Suggested implementation order

1. Solution skeleton and local Docker Compose.
2. Tenant bootstrap and identity.
3. Authentication and refresh-token lifecycle.
4. RBAC and tenant isolation.
5. Folder/document metadata and Dapper queries.
6. Versioning and MinIO integration.
7. Approval workflow and audit trail.
8. Outbox, RabbitMQ, and notification worker.
9. Cache, rate limits, resilience, and telemetry.
10. Integration/security tests and CI.
