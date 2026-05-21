# Bien ban dam phan hop dong API

- Cap dam phan: Pair 3 - Core Business -> Access Gate
- Product: A3
- Provider: Access Gate
- Consumer: Core Business
- Phien: v1.0
- Ngay: 2026-05-15

---

## Issue #1 - Thoi gian luu access log

- Ben neu van de: Consumer
- Endpoint: `GET /access/logs/recent`, `GET /access/logs/{logId}`
- Boi canh: Core Business can xem lai lich su ra vao khi co su co hoac can audit.
- Van de: Neu Access Gate luu log qua ngan, Core Business se khong du du lieu de dieu tra.
- De xuat: Access Gate luu access log toi thieu 30 ngay.
- Quyet dinh: Chap nhan.
- Rationale: Moc 30 ngay du cho tinh huong audit trong Lab 02 va van hop ly voi gia dinh luu tru cua Provider.
- Tac dong: Provider ghi ro gia dinh retention; Consumer khong mac dinh co the truy van log cu hon 30 ngay.

---

## Issue #2 - Gioi han so luong recent logs

- Ben neu van de: Provider
- Endpoint: `GET /access/logs/recent`
- Boi canh: Core Business can lay danh sach log gan day de audit nhanh.
- Van de: Neu tra ve qua nhieu log trong mot request, response co the lon va cham.
- De xuat: Them query parameter `limit`, mac dinh 20, toi da 100.
- Quyet dinh: Chap nhan.
- Rationale: Gioi han kich thuoc response giup API de du doan, de mock bang Prism va de trien khai that sau nay.
- Tac dong: Consumer can goi theo tung trang nho va dung `nextCursor` neu con du lieu.

---

## Issue #3 - Chuan hoa gia tri direction

- Ben neu van de: Consumer
- Endpoint: AccessLog schema
- Boi canh: Log ra vao can cho biet nguoi dung di vao hay di ra.
- Van de: Hai ben co the dung ten khac nhau nhu `ENTER`, `EXIT`, `IN`, `OUT`.
- De xuat: Chi dung hai gia tri `IN` va `OUT`.
- Quyet dinh: Chap nhan.
- Rationale: Enum ngan gon giup Consumer xu ly thong nhat va tranh hieu sai trong bao cao audit.
- Tac dong: Provider phai map gia tri tu thiet bi cong sang enum da thong nhat truoc khi tra response.

---

## Issue #4 - Chuan hoa trang thai access log

- Ben neu van de: Provider
- Endpoint: AccessLog schema
- Boi canh: Mot lan quet the co the thanh cong, bi tu choi, hoac loi do thiet bi/he thong.
- Van de: Neu chi tra thanh cong/that bai thi Consumer kho phan biet loi ky thuat voi tu choi nghiep vu.
- De xuat: Dung cac gia tri `ALLOWED`, `DENIED`, va `ERROR`.
- Quyet dinh: Chap nhan.
- Rationale: Ba gia tri nay tach ro truy cap thanh cong, truy cap bi tu choi va loi he thong.
- Tac dong: Consumer co the hien thi va thong ke rieng tung loai ket qua.

---

## Issue #5 - Trang thai vong doi cua card

- Ben neu van de: Consumer
- Endpoint: `GET /cards/{cardId}`
- Boi canh: Core Business can biet the con hieu luc hay khong.
- Van de: Neu Provider tra trang thai dang text tu do, Consumer kho xu ly bang logic on dinh.
- De xuat: Card status chi gom `ACTIVE`, `BLOCKED`, va `EXPIRED`.
- Quyet dinh: Chap nhan.
- Rationale: Ba trang thai nay du cho tich hop A3 hien tai va phu hop user story.
- Tac dong: Provider khong tra card status ngoai enum neu chua dam phan version moi.

---

## Issue #6 - Trang thai hoat dong cua gate

- Ben neu van de: Consumer
- Endpoint: `GET /gates/{gateId}/status`
- Boi canh: Core Business can biet cong co dang hoat dong binh thuong khong.
- Van de: Can phan biet cong bi loi bat ngo voi cong dang bao tri co ke hoach.
- De xuat: Gate status gom `ONLINE`, `OFFLINE`, va `MAINTENANCE`.
- Quyet dinh: Chap nhan.
- Rationale: `MAINTENANCE` khong nen bi hieu giong `OFFLINE` vi tac dong van hanh khac nhau.
- Tac dong: Consumer co the hien thi trang thai van hanh ro rang hon.

---

## Issue #7 - Chuan loi va truy vet request

- Ben neu van de: Provider
- Endpoint: Tat ca response loi 4xx/5xx
- Boi canh: Khi tich hop nhieu service, loi can co format thong nhat de debug.
- Van de: Neu moi endpoint tra loi theo mot kieu rieng, Consumer kho xu ly va kho trace loi.
- De xuat: Tat ca loi 4xx/5xx dung Problem Details voi `type`, `title`, `status`, `detail`, `instance`, va `correlationId`.
- Quyet dinh: Chap nhan.
- Rationale: Mot schema loi dung chung giup hai ben trace va chan doan loi nhanh hon.
- Tac dong: Moi error response trong `openapi.yaml` tham chieu schema `Problem`.

---

# Chot hop dong v1.0

Provider sign-off: Access Gate team  
Consumer sign-off: Core Business team  
Witness (GV/TA):  
Date: 2026-05-15

---

## Ghi chu warning Spectral

| Warning | Ly do chap nhan tam thoi | Ke hoach sua |
|---|---|---|
| Khong co warning nghiem trong | N/A | Tiep tuc giu `npm run lint` pass truoc khi nop |