# Consumer Analysis - Core Business (Smart Campus)

- Pair: Core Business -> Access Gate
- Role: Consumer
- Version: v1.0
- Date: 2026-05-19

## Business Analysis
Core Business consumes Access Gate APIs to support security audit, incident review, and operational visibility across campus gates.  
Consumer decisions depend on accurate log history, current gate state, and card lifecycle status.

## Functional Requirements
1. Call `GET /access/logs/recent` to fetch recent access activity for audit workflows.
2. Call `GET /access/logs/{logId}` to inspect a specific incident record.
3. Call `GET /gates/{gateId}/status` to check gate operability before actions.
4. Call `GET /cards/{cardId}` to determine card validity for business rules.
5. Parse both event variants (`CARD_SWIPE`, `MANUAL_OVERRIDE`) correctly.
6. Display nullable optional fields safely (`operatorNote`, `maintenanceReason`, `nextCursor`).

## Non-Functional Requirements
1. Reliable integration with schema-driven parsing (OpenAPI-first).
2. Fast operator experience through paged retrieval (bounded `limit`).
3. Traceability in logs using propagated `X-Correlation-Id`.
4. Resilience with retry/backoff only for retryable failures (`500`/timeout).
5. Compatibility with contract enums and field naming across releases.

## API Expectations
1. Send bearer token for all protected endpoints.
2. Send optional `X-Correlation-Id` per request for end-to-end tracing.
3. Use query filters (`from`, `to`, `limit`) for controlled access-log queries.
4. Expect success payloads from shared schemas (`AccessLogList`, `AccessLog`, `GateStatus`, `Card`).
5. Expect Problem Details for all handled error statuses.

## Security Concerns
1. Store and rotate service tokens securely; never log raw tokens.
2. Enforce least-privilege access for service account scopes.
3. Restrict exposure of sensitive access-log data in UI/export.
4. Sanitize user-visible error details to avoid information leakage.

## Error Handling
Consumer behavior by status:
- `400`: correct request data before retry.
- `401`: refresh/fix token configuration.
- `403`: stop retry and escalate as permission issue.
- `404`: show resource-not-found state and continue workflow.
- `500`: retry with backoff and include correlation id in incident log.

Consumer must always parse Problem Details fields: `type`, `title`, `status`, `detail`, `instance`, `correlationId`.

## Data Validation
1. Validate local input formats for `logId`, `gateId`, `cardId` before API call.
2. Validate `from <= to` and valid ISO date-time values.
3. Keep `limit` within provider contract range `[1,100]`.
4. Validate enums strictly and handle unknown values as integration alerts.
5. Branch event parsing by discriminator `eventType`.
6. Treat contract-defined nullable fields as optional at rendering and business-rule layers.
