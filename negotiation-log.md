# FIT4110 Lab 02 - API Contract Negotiation Log

- Negotiation Pair: Pair 3 - Core Business -> Access Gate
- Product: A3 Smart Campus
- Contract Version: v1.0
- Date: 2026-05-19
- Provider Team: Access Gate
- Consumer Team: Core Business

---

## Issue 01 - Access Log Retention Window

**Context**  
Core Business needs incident investigation data for access control disputes and security audits.

**Problem**  
Without a shared retention baseline, Consumer may request historical logs that Provider cannot guarantee.

**Proposal**  
Provider guarantees that access logs are queryable for at least 30 calendar days from event time.

**Decision**  
Accepted.

**Rationale**  
Thirty days covers the expected audit cycle in Lab 02 while keeping Provider storage assumptions realistic.

**Impact**  
Consumer must not assume logs older than 30 days are always available. Provider documents retention in API notes and operational runbook.

---

## Issue 02 - Pagination and Bounded Response Size for Recent Logs

**Context**  
`GET /access/logs/recent` is expected to be called frequently by Core dashboards and audit workflows.

**Problem**  
Unbounded result sets can increase latency, mock instability, and payload size.

**Proposal**  
Add `limit` query parameter with `default=20`, `max=100`, and cursor-style continuation through `nextCursor`.

**Decision**  
Accepted.

**Rationale**  
A bounded response improves predictability for both Prism mock and future production implementation.

**Impact**  
Consumer must paginate when reviewing long time ranges. Provider must always return `nextCursor` as either string cursor or `null`.

---

## Issue 03 - Business Status Vocabulary for Access Attempts

**Context**  
Consumer analytics requires consistent classification of outcomes for entries/exits.

**Problem**  
Different status vocabularies between teams can cause report mismatch and incorrect KPI aggregation.

**Proposal**  
Standardize `AccessLog.status` to enum: `ALLOWED`, `DENIED`, `ERROR`.

**Decision**  
Accepted.

**Rationale**  
This set clearly separates policy denial from technical faults and keeps aggregation simple.

**Impact**  
Provider maps device/vendor-specific states into the agreed enum. Consumer uses these exact values in business rules and dashboards.

---

## Issue 04 - Gate Operational State Semantics

**Context**  
Core Business monitors gate health and needs to distinguish planned interruption from incidents.

**Problem**  
Treating all unavailable states as a single value hides maintenance context and increases false alarms.

**Proposal**  
Use `GateStatus.status` enum: `ONLINE`, `OFFLINE`, `MAINTENANCE`, and include `maintenanceReason` as nullable text.

**Decision**  
Accepted.

**Rationale**  
The model preserves operational meaning without overcomplicating the contract.

**Impact**  
Consumer can suppress incident alarms during planned maintenance. Provider must populate `maintenanceReason` when status is `MAINTENANCE`.

---

## Issue 05 - Polymorphic Access Event Model

**Context**  
Access logs can originate from direct card swipes or manual security overrides.

**Problem**  
A single flat event schema cannot represent both cases cleanly and makes validation ambiguous.

**Proposal**  
Model `AccessEvent` using `oneOf` with discriminator `eventType`:
- `CARD_SWIPE` -> `CardSwipeEvent`
- `MANUAL_OVERRIDE` -> `ManualOverrideEvent`

**Decision**  
Accepted.

**Rationale**  
Discriminator-based polymorphism ensures strict validation and clear consumer parsing logic.

**Impact**  
Provider must emit payloads that match discriminator mapping exactly. Consumer must branch handling by `eventType`.

---

## Issue 06 - Standard Error Contract and Traceability

**Context**  
Both teams need fast troubleshooting during integration and demo sessions.

**Problem**  
Inconsistent error payloads make root-cause analysis slower and reduce observability.

**Proposal**  
Use RFC 9457-style Problem Details (`application/problem+json`) with fields:
`type`, `title`, `status`, `detail`, `instance`, `correlationId`.

**Decision**  
Accepted.

**Rationale**  
A uniform error envelope is simple, tool-friendly, and traceable across systems.

**Impact**  
All 4xx/5xx responses in `openapi.yaml` must reference `#/components/schemas/Problem`. Consumer logs and support scripts must include `correlationId`.

---

## Issue 07 - Authentication and Authorization Boundary

**Context**  
Access-gate data contains security-sensitive movement records.

**Problem**  
If authentication is optional or vague, unauthorized access risk increases and audit ownership becomes unclear.

**Proposal**  
Require Bearer token security globally; keep `/health` public for monitoring probes.

**Decision**  
Accepted.

**Rationale**  
This balances operational monitoring with minimum security controls for business data APIs.

**Impact**  
Consumer must send `Authorization: Bearer <token>` for protected endpoints. Provider enforces `401/403` with Problem Details on auth failures.

---

## Sign-off

| Role | Team | Representative | Signature | Date |
|---|---|---|---|---|
| Provider | Access Gate | Nguyen Minh Khang | Approved | 2026-05-19 |
| Consumer | Core Business | Tran Gia Han | Approved | 2026-05-19 |

Both parties confirm the negotiated decisions above are reflected in the current OpenAPI contract for FIT4110 Lab 02.
