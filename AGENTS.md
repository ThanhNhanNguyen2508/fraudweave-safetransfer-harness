# AGENTS.md — Quy tắc chung cho mọi agent (Claude Code, Codex, ...)

> File này tự đứng được: agent nào không đọc CLAUDE.md (ví dụ Codex) vẫn phải làm theo đủ các quy tắc ở đây.
> Dự án: **FraudWeave — phần SafeTransfer** (MoMo Tech Contest 2026). Người dùng: **TV3**, Backend & Fraud Intelligence Engineer.

## 1. Nguồn sự thật và thứ tự ưu tiên

Khi các tài liệu mâu thuẫn, tài liệu đứng trước thắng:

1. `context/decisions.md`: quyết định mới nhất có ngày và người quyết.
2. `sources/00-project-brief.md`: brief dự án. **§5 của brief thay thế §3 của contract.**
3. `sources/contract-v0.1.md`: contract chi tiết (models, endpoints, policy, fixtures).
4. `sources/pipeline-revised.md`: sơ đồ pipeline kèm ghi chú.

Nếu phát hiện mâu thuẫn chưa được phân xử thì **báo cho TV3**, ghi vào mục "Câu hỏi mở" của session-log, không tự chọn một bên rồi im lặng.

## 2. Ba thư mục (tuyệt đối không trộn lẫn)

| Tên | Đường dẫn | Chứa gì | Không chứa |
| --- | --- | --- | --- |
| **HARNESS** | `C:\Users\Dell\Downloads\MomoTechDay\Implements\Agent Workspace` | Tài liệu harness: CLAUDE.md, AGENTS.md, MAP.md, SPEC, PLAN, CONTRACT_STATUS, context/, deliverables/, sources/, .claude/ (rules, skills, agents) | Code sản phẩm, test, script chạy pipeline |
| **REPO** | `C:\Users\Dell\Downloads\MomoTechDay\Implements\Source code-SafeTransfer\FraudWaveAI-Momo` | Git repo sản phẩm: **toàn bộ** pipeline end-to-end (source, contracts, fixtures, migrations, seed, tests chính thức, scripts, docs) | Thử nghiệm nháp, output tạm, tài liệu harness, secrets đã commit |
| **SANDBOX** | `C:\Users\Dell\Downloads\MomoTechDay\Implements\Test-for-agent` | Thử nghiệm, script khám phá, benchmark nháp, output tạm | Bất cứ thứ gì pipeline hoặc CI cần để chạy |

- **Quy tắc phân xử:** "Xoá SANDBOX mà REPO vẫn chạy và CI vẫn pass" thì mới là đặt đúng chỗ.
- Test chính thức (unit, contract, golden) **phải nằm trong REPO** (`tests/`). Test nháp nằm trong SANDBOX.
- REPO không được tham chiếu tới đường dẫn HARNESS hoặc SANDBOX.
- `sources/` là **chỉ đọc**. Không sửa, không đổi tên, không xoá. Thông tin mới ghi vào `context/`.
- Tạo, xoá hoặc di chuyển file ở **bất kỳ** thư mục nào thì cập nhật `MAP.md` trong cùng phiên.

## 3. Nguyên tắc "làm được nhiều nhất có thể" (build-first, integrate-later)

- TV3 tự chủ end-to-end SafeTransfer, không chờ TV1/TV2.
- Phần phụ thuộc contract chưa chốt vẫn được làm, nhưng phải đi qua **interface/adapter + stub** để code chạy được ngay.
- Không chờ contract hoàn hảo, nhưng cũng **không tự "chốt" thay người khác**.
- Contract-first trong nội bộ TV3: Pydantic models là nguồn sự thật; OpenAPI và fixtures sinh ra từ đó.

## 4. Nhãn trạng thái contract

| Nhãn | Nghĩa | Agent được làm gì |
| --- | --- | --- |
| **FROZEN** | Đã chốt cả nhóm | Chỉ đổi qua PR vào `contracts/` có đủ 3 thành viên duyệt |
| **DRAFT-TV3** | TV3 tự quyết, có thể đổi | Được đề xuất đổi; khi đổi phải ghi `context/decisions.md`, cập nhật fixtures và báo TV1/TV2 liên quan |
| **PENDING-NEED-CONFIRM-CONTRACT** | Cần chốt với TV khác | Dùng stub/adapter; đánh marker trong code: `# PENDING-NEED-CONFIRM-CONTRACT(Kxx): <mô tả>` |

Danh sách K01–K12 và trạng thái hiện tại: `CONTRACT_STATUS.md`, **nơi duy nhất** theo dõi trạng thái contract. Quy tắc chi tiết: `.claude/rules/contract-pending.md`.

## 5. Privacy và bảo mật (bắt buộc)

- **Không bao giờ lưu hoặc log:** nội dung tin nhắn, STK thô, Discord user/guild ID, số tiền chính xác, `transfer_memo`.
- Chỉ lưu: `recipient_key` (HMAC), số đã che (`account_masked`), `amount_band`, các hash có salt.
- Secrets (`FW_HMAC_KEY`, `DATABASE_URL`, `API_KEYS`) chỉ nằm trong `REPO/.env` (không commit). `.env.example` thì commit. Không đọc, in, chép secret ra log, chat hay file khác.
- Dữ liệu demo **luôn synthetic và có cờ** (`simulation` / `mode=demo`). Dữ liệu tổ chức thật phải có `source_url`.
- Lỗi hoặc timeout **không bao giờ** trả về verdict Xanh.

## 6. Trung thực về dữ liệu và kết quả

- Không bịa dữ liệu, nguồn, số liệu, kết quả test hay trạng thái contract.
- Mọi khẳng định quan trọng gắn một trong ba mức: **[ĐÃ XÁC MINH]** (đã chạy, đã đọc nguồn) · **[GIẢ ĐỊNH]** (suy luận, cần TV3 duyệt) · **[CHƯA KIỂM CHỨNG]** (chưa có nguồn hoặc chưa chạy).
- Test fail thì báo nguyên văn output. Bước nào bỏ qua thì nói rõ là đã bỏ qua.

## 7. Git, worktree và thao tác rủi ro

Chi tiết: `context/git-workflow.md`.

- **TV3 tự commit và push (D-017).** Agent **không bao giờ** chạy `git add`, `git commit`, `git push` hay bất kỳ lệnh `gh` nào, kể cả khi được nhờ "tiện tay". Agent chỉ dùng `git status` / `git diff` / `git log` và gợi ý commit message. Chi tiết: `.claude/rules/git-permissions.md`.
- Không xoá hàng loạt, không ghi đè lịch sử, không tự tạo nhánh hay worktree. Cần thì nhắc TV3 kèm lệnh gợi ý.
- **REPO:** `main` luôn chạy được. Không commit thẳng lên `main`. Nhánh đặt tên `tv3/<module>-<mô-tả-ngắn>`, gộp qua Pull Request, commit theo Conventional Commits (`feat(vietqr): ...`). Trước push phải chạy checklist ở git-workflow §4.
- **HARNESS:** repo riêng (private), nhánh `main`. TV3 tự commit; Claude Code gợi ý message ở bước chốt phiên.
- **Chạy song song:** 1 phiên agent = 1 worktree = 1 nhánh = 1 vùng module. Worktree nằm ở `C:\Users\Dell\Downloads\MomoTechDay\Implements\worktrees\<nhánh-đổi-/-thành-->`. **TV3** tạo bằng `REPO/scripts/new_worktree.ps1` (không dùng `claude --worktree` cho REPO). Theo D-028, 26/11 chạy tuần tự; chỉ chuyển sang song song nếu tối 27/11 T-17 chưa xong. Thư mục REPO gốc giữ `main`.
- Agent luôn khởi động **từ HARNESS** và thêm worktree bằng `--add-dir`. Chỉ sửa code trong worktree của phiên mình.
- **Khoá contract:** tại một thời điểm chỉ một nhánh `tv3/contracts-*` được sửa `common/contracts.py`, `openapi.json` và schema fixtures. Ai giữ khoá thì ghi trong bảng "Worktree đang mở" ở `MAP.md`.

## 7b. Vai trò của từng agent với harness

| | Claude Code | Codex (và agent khác) |
| --- | --- | --- |
| Đọc harness, làm task trong REPO/worktree | ✅ | ✅ (đọc tay `.claude/rules/*.md` và `.claude/skills/*/SKILL.md` vì Codex không tự nạp `.claude/`) |
| Thử nghiệm trong SANDBOX | ✅ | ✅ |
| Sửa file trạng thái harness (MAP, PLAN, CONTRACT_STATUS, SPEC, session-log, `context/`, `.claude/`) | ✅ | ❌ |
| Cuối phiên | Chạy skill `close-session` | Ghi **một** file vào `deliverables/codex-inbox/` theo mẫu trong README của thư mục đó |
| Gợi ý commit message cuối phiên | ✅ | ✅ (trong ghi chú inbox) |
| `git add` / `commit` / `push`, lệnh `gh` | ❌ (TV3 tự làm) | ❌ (TV3 tự làm) |

## 8. Ngôn ngữ

- Trả lời TV3 bằng **tiếng Việt**, giữ nguyên thuật ngữ kỹ thuật tiếng Anh.
- File harness viết tiếng Việt. Tên file, thư mục, code và comment trong code viết **tiếng Anh**.

## 9. Quy trình mỗi phiên

**Đầu phiên** (trước khi trả lời câu đầu tiên):
1. Đọc `CLAUDE.md` (nếu là Claude Code), `AGENTS.md`, `MAP.md`.
2. Đọc **10 dòng cuối** của `deliverables/session-log.md` để biết việc còn dở và câu hỏi mở.
3. Nếu làm task: đọc `PLAN.md`; nếu task chạm tới contract: đối chiếu `CONTRACT_STATUS.md`.
4. Agent không đọc được `.claude/` tự động (ví dụ Codex): đọc tay các file trong `.claude/rules/` trước khi sửa REPO, và làm theo `.claude/skills/implement-task/SKILL.md`.

**Trong phiên** (mục ghi `context/` dưới đây áp dụng cho Claude Code; Codex ghi đề xuất vào inbox):
- Học được điều gì lâu dài (lý do, sở thích, ràng buộc) → ghi vào file chủ đề trong `context/`.
- Có quyết định mới → thêm một dòng vào `context/decisions.md` (ngày, quyết định, lý do, người quyết). Agent chỉ được ghi "người quyết = TV3" khi TV3 đã nói rõ; đề xuất của agent ghi trạng thái "đề xuất".

**Cuối phiên** (hoặc khi TV3 nói **"chốt phiên"**):
- **Claude Code** chạy skill `close-session`:
  1. Nhập ghi chú trong `deliverables/codex-inbox/`.
  2. Ghi **3–6 dòng** vào cuối `deliverables/session-log.md` (làm gì, file nào đổi, việc còn dở, câu hỏi mở).
  3. Cập nhật `MAP.md`, `PLAN.md`, `CONTRACT_STATUS.md`, `context/`.
  4. Kiểm tra: không có code sản phẩm trong HARNESS, không có file pipeline cần trong SANDBOX.
  5. Cho TV3 xem `git status` của HARNESS và REPO, kèm gợi ý commit message. Không commit.
- **Codex / agent khác:** không sửa file trạng thái. Chỉ tạo một file `deliverables/codex-inbox/YYYY-MM-DD-HHMM-<agent>-<slug>.md` theo mẫu, rồi nhắc TV3 mở Claude Code để "chốt phiên".
