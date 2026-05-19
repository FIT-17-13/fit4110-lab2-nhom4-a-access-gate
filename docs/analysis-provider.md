# Provider Analysis - Access Gate (Smart Campus)

- Pair: Core Business -> Access Gate
- Role: Provider
- Version: v1.0
- Date: 2026-05-19

## Business Analysis
Access Gate provides trusted operational data for Core Business to audit entry/exit activity, verify gate availability, and validate card lifecycle state.  
The provider scope is read APIs for logs, gate status, and card details, with consistent business semantics.

## Functional Requirements
1. Expose `GET /access/logs/recent` with optional time range and `limit`.
2. Expose `GET /access/logs/{logId}` for single-log investigation.
3. Expose `GET /gates/{gateId}/status` for operational monitoring.
4. Expose `GET /cards/{cardId}` for card state checks.
5. Return standardized enums:
`direction` = `IN|OUT`, `access status` = `ALLOWED|DENIED|ERROR`, `gate status` = `ONLINE|OFFLINE|MAINTENANCE`, `card status` = `ACTIVE|BLOCKED|EXPIRED`.
6. Return polymorphic `event` using `oneOf + discriminator` (`CARD_SWIPE`, `MANUAL_OVERRIDE`).

## Non-Functional Requirements
1. Contract-first OpenAPI 3.1.0 and Spectral-lint compliant.
2. Predictable payload size (`limit` default 20, max 100).
3. Observability via correlation id (`X-Correlation-Id`).
4. Backward compatibility for agreed field names and enums in v1.
5. Log retention baseline: at least 30 days for audit flow.

## API Expectations
1. Content type for success: `application/json`.
2. Content type for errors: `application/problem+json`.
3. Nullable fields represented by union type with `null` (OpenAPI 3.1 style).
4. Reusable schemas and responses via `components` + `$ref`.
5. `/health` is unauthenticated; business endpoints require bearer token.

## Security Concerns
1. Enforce Bearer token authentication on protected endpoints.
2. Enforce authorization boundaries for sensitive card/log/gate resources.
3. Avoid leaking internal infrastructure details in error `detail`.
4. Track and investigate suspicious repeated access attempts by correlation id.

## Error Handling
Use Problem Details schema for all non-2xx responses:
- `400`: invalid query/path/header format.
- `401`: missing or invalid token.
- `403`: authenticated but not authorized.
- `404`: resource not found (`logId`, `gateId`, `cardId`).
- `500`: provider internal failure or downstream read error.

Each error response includes: `type`, `title`, `status`, `detail`, `instance`, `correlationId`.

## Data Validation
1. Validate identifier patterns (`log-*`, `gate-*`, `card-*`) before processing.
2. Validate `from/to` as ISO `date-time` and reject invalid ranges.
3. Validate `limit` in `[1,100]`.
4. Validate enum values strictly and reject unknown values.
5. Validate polymorphic event payload according to discriminator mapping.
6. Allow nullable fields only where contract defines union with `null`.
