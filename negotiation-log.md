# API Contract Negotiation Log

- Negotiation pair: Pair 3 - Core Business -> Access Gate
- Product: A3
- Provider: Access Gate
- Consumer: Core Business
- Version: v1.0
- Date: 2026-05-15

---

## Issue #1 - Access log retention

- Raised by: Consumer
- Endpoint: `GET /access/logs/recent`, `GET /access/logs/{logId}`
- Concern: Core Business needs to audit access activity after incidents.
- Proposal: Access Gate keeps logs for at least 30 days.
- Resolution: Accepted
- Rationale: 30 days is enough for the Lab 02 audit scenario and keeps Provider storage assumptions realistic.
- Impact: Provider documents retention in analysis; Consumer does not assume older logs are available.

---

## Issue #2 - Recent log pagination and limit

- Raised by: Provider
- Endpoint: `GET /access/logs/recent`
- Concern: Returning too many logs can make responses slow and difficult to mock.
- Proposal: Add `limit` query parameter with default 20 and maximum 100.
- Resolution: Accepted
- Rationale: Bounded response size makes the API predictable for both Prism mock and future implementation.
- Impact: Consumer must request smaller pages and use `nextCursor` when more logs exist.

---

## Issue #3 - Direction values

- Raised by: Consumer
- Endpoint: AccessLog schema
- Concern: Consumer and Provider may use different names for entry and exit.
- Proposal: Use only `IN` and `OUT`.
- Resolution: Accepted
- Rationale: A small enum prevents ambiguity in audit reports.
- Impact: Provider maps local hardware values into the agreed enum before returning the response.

---

## Issue #4 - Access log status values

- Raised by: Provider
- Endpoint: AccessLog schema
- Concern: Access attempts can succeed, fail by policy, or fail because of device/system error.
- Proposal: Use `ALLOWED`, `DENIED`, and `ERROR`.
- Resolution: Accepted
- Rationale: These values separate business denial from technical failure.
- Impact: Consumer can display or aggregate the three outcomes separately.

---

## Issue #5 - Card lifecycle status

- Raised by: Consumer
- Endpoint: `GET /cards/{cardId}`
- Concern: Core Business needs to know why a card cannot be used.
- Proposal: Use `ACTIVE`, `BLOCKED`, and `EXPIRED`.
- Resolution: Accepted
- Rationale: These states are enough for the current A3 integration and match the user story.
- Impact: Provider must not return free-text card status values.

---

## Issue #6 - Gate operational status

- Raised by: Consumer
- Endpoint: `GET /gates/{gateId}/status`
- Concern: Core Business needs to distinguish outage from planned maintenance.
- Proposal: Use `ONLINE`, `OFFLINE`, and `MAINTENANCE`.
- Resolution: Accepted
- Rationale: Maintenance should not be treated the same as unexpected offline failure.
- Impact: Consumer can show a clearer operational state.

---

## Issue #7 - Error response and tracing

- Raised by: Provider
- Endpoint: All 4xx/5xx responses
- Concern: Inconsistent error formats make integration debugging slow.
- Proposal: Use Problem Details with `type`, `title`, `status`, `detail`, `instance`, and `correlationId`.
- Resolution: Accepted
- Rationale: A shared error schema lets both teams trace and diagnose failures consistently.
- Impact: All error responses in `openapi.yaml` reference `ProblemDetails`.

---

# Contract v1.0 Sign-off

Provider sign-off: Access Gate team  
Consumer sign-off: Core Business team  
Witness (GV/TA): Lê Trang  
Date: 2026-05-15

---

## Spectral Warnings

| Warning | Temporary acceptance reason | Fix plan |
|---|---|---|
| None expected | N/A | Keep `npm run lint` passing before submission |
