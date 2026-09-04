---
sidebar_position: 2
---

# Contract Catalog

| Attribute         | Value                                                               |
| ----------------- | ------------------------------------------------------------------- |
| **Project**       | [Project Name]                                                      |
| **Version**       | 0.1                                                                 |
| **Status**        | Draft                                                               |
| **Contract Type** | [REST API / CLI / gRPC / Event-Message / Library-SDK / combination] |

## Table of Contents

- [Contract Scope and Conventions](#contract-scope-and-conventions)
- [Contract Catalog](#contract-catalog)
- [Shared Schemas](#shared-schemas)
- [Detailed Contract Specifications](#detailed-contract-specifications)
- [Standard Error/Failure Examples](#standard-errorfailure-examples)
- [Observability](#observability)
- [Deployment Impact](#deployment-impact)
- [Source References](#source-references)

## Contract Scope and Conventions

- Base path/namespace: [e.g. `/api/v1` for REST, `mytool` as the CLI binary
  name, `myservice.v1` as the proto package, `myapp.events` as the topic
  namespace.]
- Conventions follow [Contract Design Standards](./contract-design-standards.md)
  (naming, versioning, error format, pagination).
- Roles referenced below are defined in the [Glossary](../../00-context/glossary.md).

## Contract Catalog

> One row per interface/operation. Group rows by domain with a subheading
> when the catalog grows. The **Type** column lets a project with mixed
> contracts (e.g. a REST API plus an admin CLI) document each kind in one
> table — delete rows/types that don't apply to this project.

| Type  | Operation                            | Description                          | Roles            |
| ----- | ------------------------------------ | ------------------------------------ | ---------------- |
| REST  | `POST /api/v1/[resources]`           | [Create a resource]                  | [Role]           |
| REST  | `GET /api/v1/[resources]`            | [List resources, paginated]          | [Role]           |
| REST  | `GET /api/v1/[resources]/{id}`       | [Get one resource]                   | [Role]           |
| REST  | `PATCH /api/v1/[resources]/{id}`     | [Update a resource]                  | [Role]           |
| REST  | `DELETE /api/v1/[resources]/{id}`    | [Delete/archive a resource]          | [Role]           |
| CLI   | `mytool [resources] create`          | [Create a resource]                  | [Role]           |
| gRPC  | `[Resource]Service/Create[Resource]` | [Create a resource]                  | [Role]           |
| Event | `[resource].created`                 | [Emitted when a resource is created] | [Consumers/Role] |

## Shared Schemas

### Error Response Schema

```json
{
  "error": {
    "code": "STRING_STABLE_CODE",
    "message": "Human-readable message",
    "details": [],
    "requestId": "req_12345"
  }
}
```

### Pagination Schema (Collection Responses)

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 0,
    "totalPages": 0
  }
}
```

### Status Code Matrix (REST/gRPC)

| Status | Meaning in this contract |
| ------ | ------------------------- |
| 200 | Success with response body |
| 201 | Resource created |
| 204 | Success without response body |
| 400 | Malformed request or failed validation |
| 401 | Missing/invalid credentials |
| 403 | Authenticated but not allowed |
| 404 | Resource not found |
| 409 | Conflict with current resource state |
| 422 | Semantically invalid request |
| 429 | Rate limit exceeded |
| 500 | Unexpected server error |

CLI contracts use exit codes instead — see
[Contract Design Standards](./contract-design-standards.md#error-and-failure-handling-standards).

## Detailed Contract Specifications

> Use this format for every interface/operation. Each specification states:
> type, signature, description, request/input schema + example,
> response/output schema + example, and the status/exit/result codes it can
> return.

### 1) [Contract Name]

**Type:** [REST / CLI / gRPC / Event / Library]
**Signature:** [e.g. `POST /api/v1/[resources]` (REST), `mytool [resources]
create` (CLI), `[Resource]Service/Create[Resource]` (gRPC),
`[resource].created` (Event), or `ResourceClient.create(input)` (Library)]

[What this operation does, who can call it, and key business rules.]

**Request / Input**

```json
{
  "name": "string"
}
```

**Response / Output — 201**

```json
{
  "id": "uuid",
  "name": "string",
  "createdAt": "2026-01-15T14:30:00Z"
}
```

**Status/exit/result codes:** `201`, `400`, `401`, `403`, `422`

## Standard Error/Failure Examples

> Keep one canonical example per status/code so clients can rely on the
> shape. These are the REST/gRPC/structured-output shape; CLI equivalents are
> exit codes plus a `stderr` message (see
> [Contract Design Standards](./contract-design-standards.md#error-and-failure-handling-standards)).

### 400 Bad Request

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed.",
    "details": [{ "field": "name", "issue": "required" }],
    "requestId": "req_12345"
  }
}
```

### 401 Unauthorized

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication is required.",
    "details": [],
    "requestId": "req_12345"
  }
}
```

### 404 Not Found

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

### 409 Conflict

```json
{
  "error": {
    "code": "STATE_CONFLICT",
    "message": "The operation conflicts with the current resource state.",
    "details": [],
    "requestId": "req_12345"
  }
}
```

### 422 Unprocessable Entity

```json
{
  "error": {
    "code": "UNPROCESSABLE_REQUEST",
    "message": "The request is well-formed but semantically invalid.",
    "details": [],
    "requestId": "req_12345"
  }
}
```

### 429 Too Many Requests

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Retry after the indicated period.",
    "details": [],
    "requestId": "req_12345"
  }
}
```

### 500 Internal Server Error

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred.",
    "details": [],
    "requestId": "req_12345"
  }
}
```

## Observability

- Every failure carries `requestId` (or the CLI/event equivalent correlation
  ID) for correlation with logs and error tracking.
- Domain-specific error/failure codes should be registered here as they are
  introduced.

## Deployment Impact

- Contract changes follow the versioning and deprecation rules in
  [Contract Design Standards](./contract-design-standards.md).
- Breaking changes require a new major version and a compatibility window.

## Source References

- [Contract Design Standards](./contract-design-standards.md)
- [Feature Requirements](../../01-requirements/README.md)
- [Security Architecture](../security/security-architecture.md)

---

**Last Updated**: YYYY-MM-DD
