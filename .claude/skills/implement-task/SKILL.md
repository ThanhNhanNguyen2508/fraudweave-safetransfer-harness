---
name: implement-task
description: Thực hiện một task T-xx trong PLAN.md từ đầu tới cuối (đọc task, kiểm tra contract, viết test, implement, chạy test, cập nhật harness). Dùng khi TV3 nói "làm T-xx" hoặc bắt đầu bất kỳ thay đổi code nào trong REPO.
---

# implement-task

REPO = `C:\Users\Dell\Downloads\MomoTechDay\Implements\Source code-SafeTransfer\FraudWaveAI-Momo`. HARNESS = thư mục chứa skill này.

## Quy trình

1. **Đọc task:** tìm T-xx trong `PLAN.md`: mô tả, file sẽ chạm, test chứng minh xong, phụ thuộc. Phụ thuộc chưa `done` → dừng, báo TV3. Đổi trạng thái task thành `doing`.
2. **Kiểm tra contract:** đối chiếu các Kxx liên quan trong `CONTRACT_STATUS.md` và phần tương ứng ở `SPEC.md` / `sources/contract-v0.1.md`.
   - PENDING → làm với stub/adapter + marker `# PENDING-NEED-CONFIRM-CONTRACT(Kxx): ...` (rule `contract-pending.md`).
   - Task buộc phải đổi contract DRAFT-TV3 → nói với TV3 trước khi đổi.
3. **Viết/cập nhật test** trong `REPO/tests/{unit,contract,golden}/` trước hoặc cùng lúc với code. Chạy để thấy test fail đúng lý do.
4. **Implement** phần nhỏ nhất làm test pass, theo `coding-standards.md` và `privacy-security.md`. Thử nghiệm nháp (nếu cần) → skill `sandbox-experiment`, không vào REPO.
5. **Chạy kiểm tra** trong REPO: `uv run pytest`, `uv run ruff check .`, `uv run ruff format --check .`. Fail thì sửa; không báo pass khi chưa chạy.
6. **Cập nhật harness:** `PLAN.md` (trạng thái theo `definition-of-done.md`), `MAP.md` (file mới/xoá/di chuyển), `CONTRACT_STATUS.md` (nếu thêm marker hoặc đổi DRAFT), `context/decisions.md` (nếu có quyết định), `deliverables/session-log.md`.
7. **Không commit, không push.** Cho TV3 xem `git status`, diff chính và output test.
8. **Tóm tắt đúng 5 dòng:**
   - Task: T-xx, trạng thái mới
   - Đã làm: ...
   - File đổi: ...
   - Test: lệnh + kết quả (số pass/fail)
   - Tiếp theo / câu hỏi mở: ...
