---
name: handoff-note
description: Viết ghi chú bàn giao cho TV1 hoặc TV2 (endpoint, ví dụ request/response, fixtures, việc họ cần chốt) vào deliverables/handoff/. Dùng khi một endpoint hoặc thư viện của TV3 đã sẵn sàng để đồng đội dùng, hoặc trước buổi chốt contract.
---

# handoff-note

## Bước
1. Xác định người nhận (TV1 / TV2) và các Kxx liên quan trong `CONTRACT_STATUS.md`.
2. Chỉ lấy thông tin **đã có trong REPO** (models, `openapi.json`, fixtures, test pass). Phần chưa làm thì ghi rõ "chưa có".
3. Tạo `deliverables/handoff/YYYY-MM-DD-<tv1|tv2>-<topic>.md` với các mục:
   - **Tóm tắt** (3 dòng): cái gì đã sẵn sàng, từ commit/nhánh nào.
   - **Endpoint / API**: method, path, header (`X-API-Key`, `X-Request-Id`), mã lỗi.
   - **Ví dụ request/response**: chép từ fixture thật, không tự viết số liệu.
   - **Fixtures dùng làm mock**: đường dẫn trong `contracts/fixtures/`.
   - **Cách chạy local** (lệnh `uv`), không chứa secret.
   - **Việc bạn cần chốt**: bảng Kxx · câu hỏi · hạn (bám gate).
   - **Đang là stub**: liệt kê marker PENDING liên quan.
4. Giọng văn: tiếng Việt, ngắn, đọc trong 3 phút. Không lộ STK, secret, dữ liệu thật.
5. Cập nhật `MAP.md` và session-log. Không tự gửi cho ai; TV3 tự chuyển.
