# Consumer Analysis - Pair 3 Core Business -> Access Gate

- Negotiation pair: Pair 3 - Core Business -> Access Gate
- Product: A3
- Consumer service: Core Business
- Provider service: Access Gate
- Author: A3 Access Gate team
- Date: 2026-05-15

---

## 1. Resources Needed By Consumer

| Resource | How Consumer uses it | Required fields for Consumer | Optional fields |
|---|---|---|---|
| AccessLog | Audit campus entry/exit activity | logId, cardId, gateId, direction, timestamp, status | operatorNote |
| GateStatus | Check whether a gate can be trusted for operations | gateId, status, lastHeartbeatAt | maintenanceReason |
| Card | Check whether a card is active, blocked, or expired | cardId, holderId, status, expiresAt | none |

---

## 2. APIs Consumer Needs To Call

| Method | Path | When is it called? | Expected response |
|---|---|---|---|
| GET | `/access/logs/recent` | Core Business needs recent audit data | AccessLogList |
| GET | `/access/logs/{logId}` | Core Business reviews one event | AccessLog |
| GET | `/gates/{gateId}/status` | Core Business checks gate availability | GateStatus |
| GET | `/cards/{cardId}` | Core Business checks card state | Card |

---

## 3. Error Cases Consumer Must Handle

| Status | Consumer meaning | Consumer handling |
|---:|---|---|
| 400 | Request parameter is invalid | Fix query/path value and log validation error |
| 401 | Token is missing or invalid | Refresh or configure service token |
| 403 | Core Business is not allowed to access this resource | Show authorization failure and stop retrying |
| 404 | Requested log, gate, or card does not exist | Show not found state for audit/operator flow |
| 500 | Access Gate has an internal issue | Retry with backoff and keep correlationId for support |

---

## 4. Additional Assumptions

- Core Business will send a Bearer token for all endpoints except `/health`.
- Core Business can pass `X-Correlation-Id` for tracing.
- Core Business accepts `operatorNote` and `maintenanceReason` as nullable fields.

---

## 5. Questions For Provider

1. How long are access logs retained?
2. What is the maximum safe value for `limit` on recent logs?
3. Does Access Gate return manual override activity in the same log list as card swipe activity?

---

## 6. Integration Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Provider changes field names | Core Business parsing breaks | Field names are locked in OpenAPI contract |
| Provider omits specific error detail | Operators cannot diagnose failures | All errors use ProblemDetails |
| Provider returns a new enum value without negotiation | Consumer may treat data as unknown | Breaking enum changes require version negotiation |
