---
sidebar_position: 1
---

# Contract Design Standards

| Attribute         | Value                                                               |
| ----------------- | ------------------------------------------------------------------- |
| **Project**       | [Project Name]                                                      |
| **Version**       | 0.1                                                                 |
| **Status**        | Draft                                                               |
| **Contract Type** | [REST API / CLI / gRPC / Event-Message / Library-SDK / combination] |

## Table of Contents

- [Contract Style](#contract-style)
- [Versioning Strategy](#versioning-strategy)
- [Error and Failure Handling Standards](#error-and-failure-handling-standards)
- [Payload Format Conventions](#payload-format-conventions)
- [Access Control Patterns](#access-control-patterns)
- [Rate Limiting and Backpressure](#rate-limiting-and-backpressure)
- [Observability](#observability)
- [Deployment Impact](#deployment-impact)
- [Source References](#source-references)

## Contract Style

> Adapt these to the **Contract Type** declared above. Delete the bullets that
> don't apply to this project and keep the rest concrete.

- **Primary style:** [e.g. REST JSON API for core business workflows / CLI
  command surface / gRPC services / async event schemas / a versioned
  library's public interface.]
- **GraphQL stance (REST/API projects):** [Adopt / not adopted — state the
  reason, e.g. operational and governance complexity for MVP.]
- **Naming convention:**
  - REST: plural, kebab-case nouns in paths (e.g., `/api/v1/projects`).
  - CLI: verb-first subcommands, kebab-case flags (e.g., `mytool projects create --name`).
  - gRPC/RPC: `PascalCase` service names, `PascalCase` methods (e.g.,
    `ProjectService/CreateProject`).
  - Event/Message: past-tense, dot- or slash-namespaced topics (e.g.,
    `project.created`).
  - Library/SDK: the language's idiomatic public API naming (e.g.,
    `ProjectClient.create()`).
- **Relationship/composition access:** nested routes, subcommands, or nested
  resources only when ownership is explicit (e.g., `/api/v1/projects/{projectId}/tasks`
  or `mytool projects tasks list --project-id`).
- **Action operations:** avoid verbs in REST URLs; non-CRUD actions use
  sub-resources (e.g., `/approve`, `/archive`) only when domain-specific
  behavior is required. CLI/RPC/library contracts express actions as their
  own command/method instead.

## Versioning Strategy

- **Default:** [state the scheme for this Contract Type]
  - REST: URL-based semantic major versioning (`/api/v1/...`).
  - CLI: tool semver; breaking flag/command changes bump the major version.
  - gRPC: proto package versioning (`package myservice.v1;`).
  - Event/Message: schema registry version or versioned topic name.
  - Library/SDK: package semver; breaking changes to the public API bump the
    major version.
- **Change policy:**
  - non-breaking changes (optional fields, new endpoints/commands/methods)
    stay in the same major version,
  - breaking contract changes require a new major version.
- **Deprecation window:** maintain the previous major version for at least
  one release cycle with an explicit deprecation notice.

## Error and Failure Handling Standards

- **Transport/signal semantics:** [state the mechanism]
  - REST/gRPC: standard HTTP or gRPC status codes.
  - CLI: process exit codes (`0` success, non-zero per failure class) plus a
    human-readable message on `stderr`.
  - Event/Message: dead-letter queue and retry/backoff policy for
    processing failures.
- **Minimum status code set (REST/gRPC):** `200`, `201`, `204`, `400`, `401`,
  `403`, `404`, `409`, `422`, `429`, `500`.
- **Canonical error payload (structured contracts — REST, gRPC, JSON-mode CLI
  output, event failure envelopes):**

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found.",
    "details": [],
    "requestId": "req_12345"
  }
}
```

- **Rules:**
  - `code` is stable and machine-readable,
  - `message` is human-readable and safe for clients,
  - `details` is optional structured validation/context data,
  - `requestId` is required for traceability across logs and error-tracking events.

## Payload Format Conventions

- **Structured contracts (REST, gRPC, events, JSON-mode CLI output):**
  `application/json; charset=utf-8`, `camelCase` field naming.
- **CLI text output:** human-readable by default on `stdout`; offer a
  `--json` (or similar) flag for machine-readable output using the same
  field-naming rules as structured contracts.
- **Datetime format:** ISO 8601 UTC (`YYYY-MM-DDTHH:MM:SSZ`), for example
  `2026-01-15T14:30:00Z`.
- **Boolean fields:** use `is/has/can` prefixes where meaningful (e.g., `isArchived`).
- **Collection envelope and pagination (offset-based default; applies to
  REST/gRPC list operations and paginated CLI list commands):**

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 125,
    "totalPages": 7
  }
}
```

- **Pagination defaults:** `page=1`, `pageSize=20`, maximum `pageSize=100`.

## Access Control Patterns

- **Authentication:** [Mechanism, e.g. JWT bearer tokens for an API, OS-level
  user/service-account auth for a CLI, mTLS for gRPC, signed messages for an
  event bus.] See the authentication ADR in [04-decisions](../../04-decisions/README.md).
- **Authorization model:** [e.g. hybrid RBAC + data-level policies].
  - Enforce role permissions at the contract boundary ([list roles from the
    glossary](../../00-context/glossary.md)).
  - [Data-layer enforcement mechanism, if any.]
- **Credential requirements:** [Lifetime, rotation, and revocation rules for
  tokens/keys/local credentials.]
- **Trust boundary:** the service/CLI/library validates identity and required
  claims/permissions before executing any protected operation.

## Rate Limiting and Backpressure

> Mark this section N/A if the Contract Type is a library/SDK or a
> single-user CLI with no shared backend.

- **Baseline policy (per authenticated user/IP, or per consumer for
  messaging):**
  - [N] `requests/minute` for standard read/write operations,
  - stricter limits for auth-sensitive operations ([limits for login,
    verification, password reset]).
  - Event/Message consumers: concurrency limits and backpressure strategy
    (e.g. bounded queue, consumer group scaling).
- **Limit response (REST/gRPC):** return `429 Too Many Requests` with a
  `Retry-After` header, same canonical error format with `code=RATE_LIMIT_EXCEEDED`.
- **Implementation approach:** [e.g. backend middleware with database-backed
  counters and endpoint-level guardrails.]

## Observability

- Attach `requestId` (or the CLI/event equivalent correlation ID), operation
  name, actor, and contract version as context for every failure.
- Track key contract metrics: error rate, p95 latency (or CLI command
  duration), and `429`/backpressure frequency by operation.
- Infrastructure-level metrics are monitored by the platform observability
  stack (see [Monitoring and Observability](../ops/monitoring-observability.md)).
- Alert on sustained spikes in `5xx`/failure-exit-code rates.
- Use release tagging from CI to correlate regressions with deployments.

## Deployment Impact

- Validate contract design changes on every pull request.
- Enforce backward-compatibility checks before merging breaking contract
  changes.
- Rollback strategy: revert to prior release and maintain the previous major
  contract version during the deprecation window.
- Manage environment variables/secrets for auth, rate-limiting configuration,
  and observability per environment.

## Source References

- [Architecture Solution Design](../core/architecture-solution-design.md)
- [Technology Stack](../core/technology-stack.md)
- [Contract Catalog](./contract-catalog.md)
- [Feature Requirements](../../01-requirements/README.md)
- [Security Architecture](../security/security-architecture.md)

---

**Last Updated**: YYYY-MM-DD
