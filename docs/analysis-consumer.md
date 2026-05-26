# Phan tich yeu cau - vai Consumer

- Cap phu thuoc: Pair 10 (Access Gate -> Core Business)
- Product: A3
- Consumer service: Access Gate
- Provider service: Core Business
- Ngay cap nhat: 2026-05-26

---

## 1. Du lieu Consumer can gui va nhan tu Provider

| Thuc the du lieu | Muc dich su dung | Truong bat buoc | Truong tuy chon |
|---|---|---|---|
| `AccessRequest` | Gui thong tin quet the RFID len Core Business de chay luat phe duyet | `cardId`, `gateId`, `direction`, `timestamp`, `idempotencyKey`, `operatorNote` | none |
| `AccessDecision` | Nhan ket qua phe duyet de dieu khien dong/mo cong vat ly | `decisionId`, `allow`, `reasonCode`, `policyId`, `expiresAt`, `operatorNote` | none |
| `AccessPolicy` | Nhan thong tin policy da ap dung cho decision | `policyType`, `policyId`, `name` | tuy loai policy |

---

## 2. API Consumer se goi sang Core Business

| HTTP Method | URL Path | Thoi diem kich hoat | Ket qua ky vong |
|---|---|---|---|
| POST | `/access/check` | Ngay khi dau doc RFID nhan dien duoc the tai cong | Tra ve `allow=true/false` va `reasonCode` |
| GET | `/decisions/{decisionId}` | Khi Access Gate can truy van lai ket qua da tao | Tra ve `AccessDecision` |
| GET | `/policies/access/{policyId}` | Khi Access Gate can xem policy duoc ap dung | Tra ve `AccessPolicy` voi `oneOf` + `discriminator` |

---

## 3. Error case Consumer phai xu ly

Core Business ap dung chuan RFC 7807 (`application/problem+json`). Access Gate can xu ly toi thieu cac tinh huong sau:

| HTTP Status | Nguyen nhan tu Provider | Phuong an xu ly phia Access Gate |
|---:|---|---|
| 400 | JSON sai cau truc hoac thieu `cardId`, `gateId`, `idempotencyKey` | Ghi log loi thiet bi, hien thi loi, khong mo cong |
| 401 | Bearer token het han hoac khong hop le | Log loi bao mat, refresh token neu co, giu cong dong |
| 404 | Endpoint/policy/decision khong ton tai | Bao loi cau hinh, tu choi mo cong |
| 409 | Trung `idempotencyKey` hoac xung dot request | Khong gui lap lai lien tuc, lay decision cu neu Provider ho tro |
| 500 | Core Business loi noi bo hoac qua tai | Fail-closed, giu cong dong va ghi log khan cap |
| Timeout | Core Business khong phan hoi trong 500ms | Huy request va fail-closed |

---

## 4. Gia dinh nghiep vu

- Access Gate sinh `idempotencyKey` bang `cardId + gateId + timestamp lam tron theo giay` de tranh xu ly trung khi quet the lien tuc.
- `operatorNote` dung OpenAPI 3.1 union type `type: [string, "null"]`.
- Neu the bi khoa, Core Business tra HTTP 200 voi `allow=false` va `reasonCode=CARD_BLOCKED`.
- Timeout phia Access Gate la 500ms; neu timeout thi fail-closed.
- `AccessPolicy` dung `policyType` lam discriminator voi hai loai `timeBased` va `roleBased`.

---

## 5. Cau hoi cho Provider Core Business

1. Base URL that cua Core Business khi test tich hop la gi?
2. Danh sach `reasonCode` co can bo sung ngoai contract hien tai khong?
3. Core Business co ho tro truy van lai decision bang `decisionId` khong?

---

## 6. Rui ro tich hop

| Rui ro | Tac dong | Cach xu ly |
|---|---|---|
| Core Business phan hoi cham | Cong bi treo hoac un tac | Timeout 500ms va fail-closed |
| Thieu idempotencyKey | Request trung lap tao canh bao rac | Bat buoc `idempotencyKey` trong `AccessRequest` |
| Provider tra enum moi | Access Gate parse sai | Moi enum moi phai dam phan version truoc |
