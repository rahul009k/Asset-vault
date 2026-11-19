# DocFlow

A local-first, multi-tenant document management and approval platform built to demonstrate practical .NET backend engineering.

## Project goals

- Manage tenants, users, roles, permissions, folders, documents, and document versions.
- Support a controlled document approval workflow.
- Enforce tenant isolation and resource-level authorization.
- Store document binaries separately from relational metadata.
- Publish reliable integration events using the transactional outbox pattern.
- Provide audit history, notifications, tests, and observability.

## Planned technology stack

| Area | Technology |
|---|---|
| Backend | .NET 10, ASP.NET Core |
| Database | SQL Server |
| Data access | Dapper |
| Messaging | RabbitMQ |
| Object storage | MinIO |
| Cache | Redis |
| Gateway | YARP |
| Testing | xUnit, Testcontainers |
| Logging / telemetry | Serilog, OpenTelemetry |
| Local environment | Docker Compose |
| Email testing | Mailpit |

RabbitMQ and MinIO are local learning substitutes for Azure Service Bus and Azure Blob Storage. They teach similar messaging and object-storage concepts, but are not exact service replicas.

## Architecture approach

Start with a **modular monolith** and split independently deployable services only when there is a concrete reason. Keep business logic isolated from infrastructure.

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

## Roles

| Role | Main responsibility |
|---|---|
| Tenant Admin | Tenant settings, users, invitations, roles, permissions, broad tenant administration |
| Document Manager | Folder and document lifecycle management, metadata, uploads, archive/restore |
| Editor | Create documents, edit permitted metadata, upload versions, submit for approval |
| Reviewer | Review eligible submitted versions and approve or reject with a reason |
| Viewer | Browse, preview, and download documents they are allowed to access |
| Auditor | Inspect permitted audit events and reports; audit access does not automatically grant file access |

A role is a convenient permission bundle, not the security boundary. APIs must independently enforce permissions, tenant boundaries, resource scope, and workflow state.

## Document lifecycle

```text
Draft ──submit──> Submitted ──approve──> Approved ──archive──> Archived
  ▲                    │
  └──── revise ── Rejected

Draft ──archive──> Archived
Archived ──restore──> Draft
```

Approval applies to a specific immutable document version. A reviewer cannot approve their own submission. Rejection requires a reason. Previous versions remain available according to retention and access rules.

## Security principles

- Derive tenant identity from trusted authentication context, never from an untrusted request body.
- Enforce authorization on every API; hiding a UI action is not security.
- Store invitation and refresh tokens as hashes; use one-time, expiring invitations and refresh-token rotation.
- Use secure, HttpOnly cookies and antiforgery protection when cookie authentication is used.
- Validate uploaded files using extension, declared content type, and file signature; never trust the original filename.
- Use generated object-storage keys rather than user-provided paths.
- Do not log passwords, tokens, secrets, or document contents.
- Invalidate permission/session caches when user status, role, or permission assignments change.

## Reliability principles

- Write business changes and outbox records in the same SQL transaction.
- Publish outbox messages asynchronously to RabbitMQ.
- Assume message delivery can be duplicated; consumers must be idempotent.
- SQL Server and object storage do not share a transaction. Design cleanup and reconciliation for partial failures.
- Use bounded retries, backoff, timeouts, and dead-letter handling.

## Documentation

- [Developer Documentation](docs/DEVELOPER.md)
- [User Guide](docs/USER_GUIDE.md)
- [QA Test Plan](docs/QA_TEST_PLAN.md)

## Implementation sequence

1. Confirm product rules and role/permission matrix.
2. Create solution skeleton and Docker Compose infrastructure.
3. Implement tenant bootstrap, identity, authentication, and sessions.
4. Implement RBAC, users, invitations, and permission checks.
5. Implement folders, documents, metadata, and versioning with Dapper.
6. Integrate MinIO upload/download flows.
7. Implement approval workflow, audit events, and domain rules.
8. Add outbox publishing, RabbitMQ consumers, and Mailpit notifications.
9. Add rate limiting, caching, resilience, and observability.
10. Add integration/security tests and CI.

## Project status

This repository describes the proposed project baseline. Treat routes, schemas, and workflows as design contracts until implemented and verified by tests.
