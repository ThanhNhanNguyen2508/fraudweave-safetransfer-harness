# Rule: Definition of Done

Một task T-xx chỉ được đánh `done` khi **tất cả** điều sau đúng:

1. Test chứng minh xong (ghi trong PLAN.md) nằm trong `REPO/tests/` và **pass** (`uv run pytest`), output đã cho TV3 xem.
2. `uv run ruff check .` và `uv run ruff format --check .` sạch.
3. Marker PENDING (nếu có) đúng định dạng và khớp `CONTRACT_STATUS.md`.
4. Không có secret, STK thô hay dữ liệu thật trong code/fixtures/log.
5. `PLAN.md` đã cập nhật trạng thái; `MAP.md` đã cập nhật nếu có file tạo/xoá/di chuyển.
6. `deliverables/session-log.md` đã ghi.
7. Đổi contract DRAFT-TV3 → đã làm đủ 4 bước trong `contract-pending.md`.

Chưa đủ thì trạng thái là `doing`, và nói rõ còn thiếu bước nào. Không bao giờ báo "done" dựa trên test chưa chạy.
