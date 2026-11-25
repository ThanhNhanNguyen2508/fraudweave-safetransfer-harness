# Rule: làm việc với contract chưa chốt

Trạng thái K01–K12 chỉ theo dõi ở `CONTRACT_STATUS.md` (HARNESS).

## PENDING-NEED-CONFIRM-CONTRACT
- Mọi điểm phụ thuộc contract PENDING phải đi qua **interface/adapter + stub**. Logic SafeTransfer gọi interface, không gọi thẳng dữ liệu/định dạng của TV1/TV2.
- Stub phải chạy được, có test, trả dữ liệu theo bản nháp trong contract v0.1.
- Đánh marker ngay tại chỗ, **đúng định dạng** (một dòng, ID hai chữ số):
  ```python
  # PENDING-NEED-CONFIRM-CONTRACT(K07): screening_id lookup is a stub until TV1 confirms ClaimContext fields
  ```
- Thêm marker mới → cập nhật cột "Stub/adapter trong code" trong `CONTRACT_STATUS.md`.
- **Không tự chốt thay người khác.** Agent không được đổi trạng thái PENDING → DRAFT/FROZEN; chỉ TV3 làm, sau khi bên liên quan xác nhận.

## DRAFT-TV3
Khi đổi một contract DRAFT-TV3 (field, enum, reason code, luật policy), trong **cùng một thay đổi** phải:
1. Cập nhật `CONTRACT_STATUS.md` (cột ngày + dòng lịch sử).
2. Thêm dòng vào `context/decisions.md` (ngày, gì, vì sao, ai quyết).
3. Cập nhật fixtures trong `REPO/contracts/fixtures/` và test tương ứng; sinh lại `openapi.json`.
4. Ghi vào session-log rằng cần báo TV1/TV2 liên quan.

## FROZEN
- Chỉ đổi qua PR vào `contracts/` có cả 3 thành viên duyệt (contract §10).
- Additive (field optional mới, reason code mới, enum mới ở cuối): một approval từ mỗi consumer, không bump version.
- Breaking (rename, xoá, đổi kiểu/nghĩa): họp nhóm, bump `contract_version`, cập nhật fixtures cùng PR.
