# Rule: đặt file đúng thư mục

**Quy tắc phân xử:** "Xoá SANDBOX mà REPO vẫn chạy và CI vẫn pass" thì mới là đặt đúng chỗ.

| Loại file | HARNESS | REPO | SANDBOX |
| --- | --- | --- | --- |
| CLAUDE.md, AGENTS.md (harness), MAP, SPEC, PLAN, CONTRACT_STATUS (bản chính) | ✅ | | |
| `context/`, `deliverables/` (báo cáo, handoff, session log), `.claude/` (rules, skills, agents) | ✅ | | |
| `sources/` (tài liệu thô của TV3, **chỉ đọc**) | ✅ | | |
| Source code `src/`, `web/`, migrations, seed synthetic, scripts pipeline | | ✅ | |
| Contracts: Pydantic, `openapi.json`, fixtures (normalize, qr, transfer, eval), bản sao CONTRACT_STATUS | | ✅ | |
| **Test chính thức** (unit, contract, golden) → `REPO/tests/` | | ✅ | |
| `pyproject.toml`, `.gitignore`, `.env.example`, README, AGENTS.md ngắn cho đồng đội, `docs/` | | ✅ | |
| `.env` thật (không commit) | | ✅ | |
| Test nháp, benchmark thử, notebook khám phá, script one-off, output tạm, ảnh QR thử | | | ✅ |

Quy tắc kèm theo:
- REPO **không được** import, đọc hay tham chiếu đường dẫn tới SANDBOX hoặc HARNESS.
- Thứ gì trong SANDBOX chứng minh được giá trị → **viết lại sạch** vào REPO (kèm test), không copy nguyên file nháp.
- HARNESS không chứa code chạy được của sản phẩm. Đoạn code minh hoạ ngắn trong tài liệu thì được.
- Mỗi thử nghiệm trong SANDBOX là một thư mục `YYYY-MM-DD-<slug>/` có README 3 dòng (skill `sandbox-experiment`).
- Tạo, xoá, di chuyển file ở bất kỳ thư mục nào → cập nhật `MAP.md`.
