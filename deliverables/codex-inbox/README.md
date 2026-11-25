# codex-inbox — ghi chú cuối phiên của Codex (và agent khác ngoài Claude Code)

Agent không phải Claude Code **không sửa file trạng thái của harness** (MAP, PLAN, CONTRACT_STATUS, SPEC, session-log, context/, .claude/).
Cuối phiên, agent đó tạo **một file** trong thư mục này. Claude Code đọc và nhập vào harness ở bước "chốt phiên" (skill `close-session`), xong thì chuyển file sang `processed/`.

Tên file: `YYYY-MM-DD-HHMM-<agent>-<slug>.md`, ví dụ `2026-11-27-1430-codex-vietqr-crc.md`.

Mẫu nội dung:

```markdown
# <agent> · <ngày giờ> · <T-xx>
- Worktree / nhánh: <path> · <branch>
- Đã làm: ...
- File đổi trong REPO: ...
- Test: <lệnh> → <kết quả nguyên văn, số pass/fail>
- Đề xuất cập nhật harness: PLAN (T-xx → trạng thái), MAP (file mới/xoá), CONTRACT_STATUS (marker Kxx), decisions (nếu có)
- Việc còn dở: ...
- Câu hỏi mở: ...
```
