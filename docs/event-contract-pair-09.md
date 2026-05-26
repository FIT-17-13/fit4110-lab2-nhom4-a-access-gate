# Event Contract - Pair 9 Access Gate -> Analytics

- Pair: 9
- Producer: Access Gate
- Consumer: Analytics
- Mechanism: Queue async; Lab 03 mock by `POST /events`
- Event type: `access.logs.created`
- Event version: `1.0.0`
- Source: `access-gate`
- Idempotency key: `eventId`

---

## 1. Event Envelope

Analytics confirms that Access Gate will publish the access log event using this shared envelope:

```json
{
  "eventId": "evt-20260526-0001",
  "eventType": "access.logs.created",
  "eventVersion": "1.0.0",
  "occurredAt": "2026-05-26T08:00:00Z",
  "source": "access-gate",
  "correlationId": "corr-20260526-0001",
  "data": {
    "logId": "log-7788",
    "cardId": "RFID-2026-001",
    "gateId": "gate-main-01",
    "direction": "IN",
    "status": "ALLOWED"
  }
}
```

---

## 2. Required Fields

| Field | Type | Required | Description |
|---|---|---:|---|
| `eventId` | string | yes | Unique event id, used by Analytics for idempotency |
| `eventType` | string | yes | Must be `access.logs.created` |
| `eventVersion` | string | yes | Must be `1.0.0` |
| `occurredAt` | string date-time | yes | Time when the access event occurred |
| `source` | string | yes | Must be `access-gate` |
| `correlationId` | string | yes | Trace id across services |
| `data.logId` | string | yes | Access log id |
| `data.cardId` | string | yes | RFID card id |
| `data.gateId` | string | yes | Gate id |
| `data.direction` | string | yes | Access direction |
| `data.status` | string | yes | Access result |

---

## 3. Enums

```text
direction: IN, OUT
status: ALLOWED, DENIED, ERROR
```

---

## 4. Optional Fields

Analytics accepts these fields when Access Gate has the data:

| Field | Type | Description |
|---|---|---|
| `data.personId` | string | Preferred general person identifier |
| `data.studentId` | string | Allowed if Access Gate only has student id |
| `data.buildingId` | string | Building where the gate is located |
| `data.readerId` | string | Physical reader id |
| `data.reasonCode` | string | Reason code when status is `DENIED` or `ERROR` |
| `data.riskLevel` | string | Optional risk level: `low`, `medium`, `high`, `critical` |

Analytics prefers `personId` over `studentId` because `personId` can represent students, staff, visitors, or unknown people.

---

## 5. Lab 03 Mock Endpoint

Analytics will mock event ingestion by HTTP first:

```http
POST /events
```

Success response:

```http
202 Accepted
```

```json
{
  "accepted": true,
  "eventId": "evt-20260526-0001",
  "status": "accepted",
  "message": "Event accepted for analytics processing"
}
```

Validation error for missing required fields:

```http
400 Bad Request
Content-Type: application/problem+json
```

```json
{
  "type": "https://example.com/problems/bad-request",
  "title": "Bad Request",
  "status": 400,
  "detail": "Missing required field: data.gateId",
  "instance": "/events"
}
```

Semantic validation error for invalid enum values:

```http
422 Unprocessable Entity
Content-Type: application/problem+json
```

```json
{
  "type": "https://example.com/problems/unprocessable-entity",
  "title": "Unprocessable Entity",
  "status": 422,
  "detail": "direction must be IN or OUT; status must be ALLOWED, DENIED or ERROR",
  "instance": "/events"
}
```

---

## 6. Real Broker Integration Later

RabbitMQ/Kafka/MQTT, host, port, retry policy, dead-letter queue, and ack strategy are recorded for real integration after the Lab 03 mock flow is stable.

Analytics will derive metrics from this event:

- total access count;
- count by `IN` and `OUT`;
- count by `ALLOWED`, `DENIED`, and `ERROR`;
- count by `gateId`;
- count by time range `from` and `to`.
