# Bien ban dam phan hop dong API

- Product: A3 - Access Gate
- Pham vi: Pair 3 va Pair 10
- Phien: v1.0
- Ngay cap nhat: 2026-05-26

---

# Phan A - Pair 3: Core Business -> Access Gate

- Provider: Access Gate
- Consumer: Core Business
- Muc dich: Core Business truy van log quet the, trang thai cong va thong tin the tu Access Gate.

## Issue #1 - Endpoint truy van log gan day

- Raised by: Consumer
- Endpoint: `GET /access/logs/recent`
- Concern: Core Business can lay log gan day de audit nhung khong muon response qua lon.
- Proposal: Ho tro `limit` va `cursor`, mac dinh `limit=20`, toi da 100.
- Resolution: Accepted.
- Rationale: Phan trang giup API on dinh va de mock/test.
- Impact: Consumer phai dung `nextCursor` neu can lay them du lieu.

## Issue #2 - Chuan hoa dinh dang ma the va ma cong

- Raised by: Provider
- Endpoint: Tat ca endpoint Pair 3
- Concern: Pair 3 va Pair 10 can dung chung format de ket noi that.
- Proposal: `cardId` dung dang `RFID-2026-001`, `gateId` dung dang `gate-main-01`.
- Resolution: Accepted.
- Rationale: Giam loi mapping giua Access Gate, Core Business va Analytics.
- Impact: Cac response Access Gate phai tra dung format da chot.

## Issue #3 - Chuan hoa status cua access log

- Raised by: Consumer
- Endpoint: AccessLogItem schema
- Concern: Ben Core co the dung `SUCCESS`, trong khi Access Gate can phan biet ro ket qua truy cap.
- Proposal: Dung `ALLOWED`, `DENIED`, `ERROR`.
- Resolution: Accepted, voi ghi chu neu Core bat buoc dung `SUCCESS` thi can version moi.
- Rationale: `ALLOWED` khop voi nghiep vu mo cong hon `SUCCESS` va dong bo voi Pair 10 `allow=true`.
- Impact: Consumer can parse enum theo contract.

## Issue #4 - Trang thai gate va card

- Raised by: Provider
- Endpoint: `GET /gates/{gateId}/status`, `GET /cards/{cardId}`
- Concern: Can enum on dinh de Core Business xu ly.
- Proposal: Gate status gom `ONLINE`, `OFFLINE`, `MAINTENANCE`; Card status gom `ACTIVE`, `BLOCKED`, `EXPIRED`.
- Resolution: Accepted.
- Rationale: Du cho cac tinh huong van hanh va audit trong Lab 2.
- Impact: Provider khong tra enum ngoai danh sach neu chua dam phan.

---

# Phan B - Pair 10: Access Gate -> Core Business

- Provider: Core Business
- Consumer: Access Gate
- Muc dich: Access Gate goi Core Business realtime de kiem tra policy truoc khi mo cong.

## Issue #5 - Timeout va co che fail-closed

- Raised by: Access Gate (Consumer)
- Endpoint: `POST /access/check`
- Concern: Access Gate can phan hoi nhanh de khong un tac tai cong.
- Proposal: Core Business phan hoi muc tieu 200ms; Access Gate timeout sau 500ms.
- Resolution: Accepted.
- Rationale: Core Business can chay nhieu lop policy nen 100ms khong on dinh; 200ms la muc tieu hop ly.
- Impact: Access Gate cai timeout 500ms va fail-closed neu qua thoi gian.

## Issue #6 - Idempotency cho quet the lien tuc

- Raised by: Core Business (Provider)
- Endpoint: `POST /access/check`
- Concern: Mot the co the bi quet nhieu lan trong thoi gian ngan, gay request trung lap.
- Proposal: Access Gate gui `idempotencyKey` trong `AccessRequest`.
- Resolution: Accepted.
- Rationale: Provider co the nhan dien request trung lap va tranh sinh canh bao rac.
- Impact: `idempotencyKey` la truong bat buoc trong contract.

## Issue #7 - operatorNote co the null

- Raised by: Core Business (Provider)
- Endpoint: `POST /access/check`
- Concern: Ghi chu cua operator khong phai luc nao cung co.
- Proposal: Dung OpenAPI 3.1 union type `type: [string, "null"]` cho `operatorNote`.
- Resolution: Accepted.
- Rationale: Mo ta dung ban chat du lieu, khong can dung chuoi rong.
- Impact: Access Gate phai xu ly duoc ca string va null.

## Issue #8 - The bi khoa la nghiep vu binh thuong

- Raised by: Access Gate (Consumer)
- Endpoint: `POST /access/check`
- Concern: The bi khoa nen la ket qua nghiep vu, khong nen la HTTP error.
- Proposal: Tra HTTP 200 voi `allow=false`, `reasonCode=CARD_BLOCKED`.
- Resolution: Accepted.
- Rationale: Giup Access Gate hien thi ly do tu choi muot hon va khong xem la loi he thong.
- Impact: Consumer khong throw exception cho `CARD_BLOCKED`.

## Issue #9 - Chuan loi Problem Details

- Raised by: Core Business (Provider)
- Endpoint: Tat ca API
- Concern: Can format loi thong nhat de Consumer xu ly.
- Proposal: Tat ca loi 4xx/5xx dung RFC 7807 Problem Details.
- Resolution: Accepted.
- Rationale: Phu hop OpenAPI contract va tieu chi Lab 2.
- Impact: Tat ca responses loi tham chieu schema `Problem`.

## Issue #10 - Policy polymorphism

- Raised by: Core Business (Provider)
- Endpoint: `GET /policies/access/{policyId}`
- Concern: He thong co nhieu loai policy nhu theo gio va theo vai tro.
- Proposal: Dung `oneOf` va `discriminator`, truong `policyType` map voi `timeBased` va `roleBased`.
- Resolution: Accepted.
- Rationale: Consumer biet chinh xac dang nhan loai policy nao.
- Impact: Access Gate parse JSON linh hoat theo `policyType`.

---

# Chot hop dong v1.0

Provider Pair 3 sign-off: Access Gate team  
Consumer Pair 3 sign-off: Core Business team  
Provider Pair 10 sign-off: Core Business team  
Consumer Pair 10 sign-off: Access Gate team  
Witness (GV/TA):  
Date: 2026-05-26

---

## Ghi chu Lab 3

Pair 9 (Access Gate -> Analytics) la queue async. Lab 2 ghi nhan so bo de chuyen sang Lab 3 voi topic de xuat `access.logs.created`.
