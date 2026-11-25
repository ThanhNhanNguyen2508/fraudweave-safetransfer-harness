---
name: close-session
description: Chốt phiên. Chỉ Claude Code chạy skill này. Nhập ghi chú của Codex trong codex-inbox, cập nhật session-log, MAP, PLAN, CONTRACT_STATUS, decisions, chạy contract-audit, rồi cho TV3 xem git status và gợi ý commit message (TV3 tự commit). Dùng khi TV3 nói "chốt phiên" hoặc khi kết thúc một phiên làm việc.
---

# close-session

Claude Code là agent **duy nhất** cập nhật trạng thái harness cuối phiên (D-009).

## Bước
1. **Nhập inbox:** đọc mọi file `deliverables/codex-inbox/*.md` (không tính `README.md`, `processed/`). Đối chiếu từng đề xuất với REPO thật (`git -C <worktree> status/log`, chạy lại test nếu ghi chú có số liệu). Không tin nguyên văn: ghi chú nào chưa kiểm được thì gắn [CHƯA KIỂM CHỨNG].
2. **Cập nhật harness** dựa trên phiên này cộng với inbox đã kiểm:
   - `PLAN.md`: trạng thái T-xx theo `.claude/rules/definition-of-done.md`.
   - `MAP.md`: file tạo/xoá/di chuyển ở cả 3 thư mục, bảng "Worktree đang mở".
   - `CONTRACT_STATUS.md`: marker hoặc stub mới, trạng thái đổi (chỉ khi TV3 đã xác nhận).
   - `context/decisions.md` và file chủ đề trong `context/` nếu có điều học được lâu dài.
3. **contract-audit:** chạy skill `contract-audit` nếu REPO có thay đổi code trong phiên.
4. **session-log:** thêm 3–6 dòng vào cuối `deliverables/session-log.md`: làm gì (ghi cả phần Codex làm), file đổi, việc còn dở, câu hỏi mở.
5. **Dọn inbox:** chuyển các file đã nhập sang `deliverables/codex-inbox/processed/`.
6. **Kiểm tra tự thân:** HARNESS không có code sản phẩm; SANDBOX không chứa thứ pipeline cần; `git -C <HARNESS> status`.
7. **Gợi ý commit (D-017):** cho TV3 xem `git status` của HARNESS và REPO/worktree liên quan, kèm commit message gợi ý (`docs(harness): <tóm tắt>`; với REPO thì theo Conventional Commits). **Không** chạy `git add` / `commit` / `push` / `gh`; TV3 tự làm.
8. Cuối cùng liệt kê câu hỏi cần hỏi TV1/TV2 (nếu có).
