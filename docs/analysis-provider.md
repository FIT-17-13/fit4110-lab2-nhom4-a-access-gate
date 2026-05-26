# Phan tich yeu cau - vai Provider

- Cap phu thuoc: Pair 3 (Core Business -> Access Gate)
- Product: A3
- Provider service: Access Gate
- Consumer service: Core Business
- Ngay cap nhat: 2026-05-26

---

## 1. Resource chinh Access Gate cung cap

| Resource | Mo ta | Truong bat buoc | Truong tuy chon |
|---|---|---|---|
| `AccessLogItem` | Mot ban ghi lich su quet the tai cong | `logId`, `cardId`, `gateId`, `direction`, `timestamp`, `status` | `operatorNote` |
| `GateStatus` | Trang thai hoat dong cua cong | `gateId`, `status`, `lastHeartbeatAt` | `maintenanceReason` |
| `Card` | Thong tin the RFID trong he thong Access Gate | `cardId`, `holderId`, `status`, `expiresAt` | none |

---

## 2. API Provider cung cap cho Core Business

| Method | Path | Muc dich | Khi nao Consumer goi? |
|---|---|---|---|
| GET | `/access/logs/recent` | Lay danh sach log quet the gan day | Core Business can audit hoat dong ra vao |
| GET | `/access/logs/{logId}` | Lay chi tiet mot access log | Core Business can dieu tra mot su kien cu the |
| GET | `/gates/{gateId}/status` | Lay trang thai cong | Core Business can biet cong online/offline/maintenance |
| GET | `/cards/{cardId}` | Lay thong tin the | Core Business can kiem tra the active/blocked/expired |

---

## 3. Error case Provider tra ve

| Status | Tinh huong | Response body |
|---:|---|---|
| 400 | Query/path parameter sai dinh dang | `Problem` |
| 401 | Thieu hoac sai Bearer token | `Problem` |
| 403 | Core Business khong co quyen truy van resource | `Problem` |
| 404 | Khong tim thay log, gate hoac card | `Problem` |
| 500 | Access Gate loi doc log store hoac gate registry | `Problem` |

---

## 4. Gia dinh bo sung

- `cardId` dung dinh dang thong nhat `RFID-2026-001` de khop voi Pair 10 va event Pair 9.
- `gateId` dung dinh dang `gate-main-01`.
- `direction` chi co `IN` hoac `OUT`.
- `status` cua access log chi co `ALLOWED`, `DENIED`, `ERROR`.
- Tat ca loi 4xx/5xx dung `application/problem+json` va schema `Problem`.

---

## 5. Cau hoi can chot voi Core Business

1. Core Business co can filter log theo khoang thoi gian `from/to` khong?
2. Core Business co chap nhan status `ALLOWED` thay vi `SUCCESS` khong?
3. Core Business co can them field `buildingId` hoac `readerId` trong AccessLogItem khong?

---

## 6. Rui ro tich hop

| Rui ro | Tac dong | Cach xu ly |
|---|---|---|
| Pair 3 va Pair 10 dung format `cardId` khac nhau | Mapping du lieu loi khi ket noi that | Chuan hoa `cardId` thanh `RFID-YYYY-NNN` |
| Consumer mong `SUCCESS` nhung Provider tra `ALLOWED` | Core Business parse/logic sai | Chot enum trong negotiation log truoc khi code |
| Response log qua lon | Cham API va kho mock | Dung `limit` toi da 100 va `nextCursor` |
