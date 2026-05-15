# Provider Analysis - Pair 3 Core Business -> Access Gate

- Negotiation pair: Pair 3 - Core Business -> Access Gate
- Product: A3
- Provider service: Access Gate
- Consumer service: Core Business
- Author: A3 Access Gate team
- Date: 2026-05-15

---

## 1. Main Resources

| Resource | Description | Required attributes | Optional attributes |
|---|---|---|---|
| AccessLog | A card swipe or manual gate access record | logId, cardId, gateId, direction, timestamp, status | operatorNote |
| GateStatus | Current operational state of a gate | gateId, status, lastHeartbeatAt | maintenanceReason |
| Card | Card identity and lifecycle status | cardId, holderId, status, expiresAt | none |

---

## 2. Expected APIs

| Method | Path | Purpose | When does Consumer call it? |
|---|---|---|---|
| GET | `/access/logs/recent` | Return recent access logs | Core Business needs audit data |
| GET | `/access/logs/{logId}` | Return one access log | Core Business investigates one access event |
| GET | `/gates/{gateId}/status` | Return current gate status | Core Business checks whether a gate is online |
| GET | `/cards/{cardId}` | Return card information | Core Business checks whether a card is active, blocked, or expired |

---

## 3. Error Cases

| Status | Situation | Expected response body |
|---:|---|---|
| 400 | Query parameter has invalid format | ProblemDetails |
| 401 | Bearer token is missing or invalid | ProblemDetails |
| 403 | Caller has no permission for the requested gate/card/log | ProblemDetails |
| 404 | logId, gateId, or cardId does not exist | ProblemDetails |
| 500 | Access Gate cannot read log store or gate registry | ProblemDetails |

---

## 4. Additional Assumptions

- Access logs are retained for at least 30 days.
- `/access/logs/recent` defaults to 20 items and allows a maximum of 100 items.
- All error responses use `application/problem+json` and include a `correlationId`.
- Gate status values are limited to `ONLINE`, `OFFLINE`, and `MAINTENANCE`.
- Card status values are limited to `ACTIVE`, `BLOCKED`, and `EXPIRED`.

---

## 5. Questions For Consumer

1. Does Core Business need time range filtering for recent logs?
2. Should Core Business receive manual override events in the same access log response?
3. Which teams are allowed to query blocked or expired card details?

---

## 6. Integration Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Consumer and Provider use different status names | Core Business may parse the response incorrectly | Status enums are fixed in `openapi.yaml` |
| Recent log response is too large | Slow response or mock failure | Limit defaults to 20 and maximum is 100 |
| Missing correlation id | Harder to trace integration errors | Support optional `X-Correlation-Id` request header and response ProblemDetails correlationId |
