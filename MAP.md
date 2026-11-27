# MAP — bản đồ 3 thư mục

> Cập nhật mỗi khi tạo, xoá hoặc di chuyển file ở **bất kỳ** thư mục nào. Lần cuối: 2026-11-27. Chỉ Claude Code sửa file này; Codex đề xuất qua `deliverables/codex-inbox/`.
> Gốc: `C:\Users\Dell\Downloads\MomoTechDay\Implements\`

## Tìm / lưu thông tin ở đâu

| Muốn… | Tìm ở | Lưu vào |
| --- | --- | --- |
| Biết dự án, mục tiêu, phân công gốc | `sources/00-project-brief.md` | — (chỉ đọc) |
| Phạm vi, decision table, golden cases, Done theo gate | `SPEC.md` | `SPEC.md` (+ dòng trong `context/decisions.md`) |
| Việc cần làm, trạng thái task | `PLAN.md` | `PLAN.md` |
| Trạng thái contract K01–K12 | `CONTRACT_STATUS.md` | `CONTRACT_STATUS.md` (nơi duy nhất) |
| Chi tiết models/endpoint/policy | `sources/contract-v0.1.md` | Đổi contract → theo `.claude/rules/contract-pending.md` |
| Vì sao làm vậy | `context/decisions.md` | `context/decisions.md` |
| Ai làm gì | `context/team-and-ownership.md` | như cũ |
| Thuật ngữ | `context/glossary.md` | như cũ |
| Kiến thức lâu dài khác | `context/<chủ-đề>.md` | tạo file chủ đề mới + thêm dòng vào MAP |
| Phiên trước làm gì | `deliverables/session-log.md` (10 dòng cuối) | cuối `session-log.md` |
| Báo cáo, ghi chú bàn giao | `deliverables/` | `deliverables/` (`handoff/` cho TV1/TV2) |
| Code, test chính thức, fixtures | REPO | REPO |
| Thử nghiệm nháp | SANDBOX | SANDBOX `YYYY-MM-DD-<slug>/` |
| Branch, commit, worktree, checklist push, đồng bộ máy | `context/git-workflow.md` | như cũ |
| Codex làm gì trong phiên | `deliverables/codex-inbox/` | Codex ghi file mới; Claude Code nhập lúc chốt phiên |
| Worktree nào đang mở, ai giữ khoá contract | §4 file này | §4 file này |

## 1. HARNESS — `Agent Workspace\`

Git repo riêng, nhánh `main`, chưa có commit và chưa có remote (đợi TV3 duyệt push lần đầu). Gồm 3 tầng: nền tảng (AGENTS/CLAUDE/MAP/context/deliverables/sources), vận hành (SPEC/PLAN/CONTRACT_STATUS/.claude), đồng bộ và song song (git, git-workflow, codex-inbox).

| Đường dẫn | Nội dung |
| --- | --- |
| `CLAUDE.md` | Điểm vào cho Claude Code: TV3 là ai, dự án, cách làm việc; import AGENTS, MAP, rules |
| `CLAUDE.local.md` | Sở thích cá nhân của TV3 (gitignored) |
| `AGENTS.md` | Quy tắc chung cho mọi agent (Claude Code, Codex): 3 thư mục, nhãn contract, privacy, quy trình phiên |
| `MAP.md` | File này |
| `README.md` | Hướng dẫn bắt đầu ngắn cho người mới / Codex: 3 tầng, lệnh khởi động 2 agent |
| `SPEC.md` | Phạm vi, mục tiêu đo được, decision table, 11 golden cases, Done theo gate |
| `PLAN.md` | Task T-01…T-36 (có T-04a/b, T-06b/c/d, T-07a) bám gate G1/G2/G3, đường găng G1, trạng thái, phụ thuộc, test chứng minh |
| `CONTRACT_STATUS.md` | Bảng K01–K12 + stub/adapter + câu hỏi mở + lịch sử thay đổi |
| `.gitignore` | Loại CLAUDE.local.md, settings.local.json, `.claude/worktrees/`, .env*, thư mục/file tạm |
| `.claude/settings.json` | additionalDirectories (REPO, SANDBOX, worktrees), allow lệnh an toàn, ask reset/clean/xoá, **deny** `git add/commit/push`, `gh`, đọc/sửa `.env`, sửa `sources/` |
| `.claude/rules/folder-placement.md` | File nào đặt ở HARNESS / REPO / SANDBOX |
| `.claude/rules/contract-pending.md` | Cách làm với PENDING / DRAFT-TV3 / FROZEN, định dạng marker |
| `.claude/rules/privacy-security.md` | Cái gì không bao giờ lưu, secrets, dữ liệu synthetic |
| `.claude/rules/coding-standards.md` | Python 3.11, FastAPI, Pydantic v2, ruff, pytest, type hints |
| `.claude/rules/definition-of-done.md` | Điều kiện để task được đánh `done` |
| `.claude/rules/git-permissions.md` | Agent chỉ được `git status/diff/log`; không add/commit/push/gh (D-017) |
| `.claude/rules/model-routing.md` | Phân vai Sonnet/Opus, task rủi ro cao, không tự đổi model (D-030) |
| `.claude/skills/implement-task/SKILL.md` | Quy trình làm một task T-xx |
| `.claude/skills/contract-audit/SKILL.md` | Đối chiếu marker PENDING trong REPO với CONTRACT_STATUS |
| `.claude/skills/add-golden-case/SKILL.md` | Thêm golden case (fixture + seed + test) |
| `.claude/skills/handoff-note/SKILL.md` | Viết ghi chú bàn giao cho TV1/TV2 vào `deliverables/handoff/` |
| `.claude/skills/sandbox-experiment/SKILL.md` | Thử nghiệm nháp trong SANDBOX |
| `.claude/skills/close-session/SKILL.md` | Chốt phiên (chỉ Claude Code): nhập codex-inbox, cập nhật harness, cho xem git status và gợi ý commit message (TV3 tự commit) |
| `.claude/agents/code-reviewer.md` | Subagent review chỉ đọc |
| `.claude/agents/contract-auditor.md` | Subagent kiểm tra contract trước gate, chỉ đọc |
| `context/decisions.md` | Log quyết định D-001…, có ngày, lý do, người quyết |
| `context/team-and-ownership.md` | Thành viên, module TV3 sở hữu, điểm giao với TV1/TV2 |
| `context/glossary.md` | Thuật ngữ: Claim, C1–C9, verdict, recipient_key, tier, evidence... |
| `context/git-workflow.md` | Branch/commit REPO, .gitignore đề xuất, checklist push, quy tắc worktree, chép .env, đồng bộ máy |
| `deliverables/session-log.md` | Nhật ký phiên (append ở cuối) |
| `deliverables/codex-inbox/README.md` | Quy ước và mẫu ghi chú cuối phiên của Codex; file đã nhập chuyển sang `processed/` |
| `sources/00-project-brief.md` | Brief dự án, nguồn sự thật (chỉ đọc) |
| `sources/contract-v0.1.md` | Integration Contract v0.1; §3 đã bị brief §5 thay thế (chỉ đọc) |
| `sources/pipeline-revised.md` | Sơ đồ Mermaid pipeline + ghi chú sau phân công mới (chỉ đọc) |
| `sources/01-harness-prompts.md` | Bộ prompt dựng harness 4 bước (chỉ đọc) |

## 2. REPO — `Source code-SafeTransfer\FraudWaveAI-Momo\`

Git repo sản phẩm, remote `github.com/ThanhNhanNguyen2508/FraudWaveAI-Momo` (tài khoản của TV3). Nhánh làm việc `tv3/scaffold-repo` (TV3 tạo). T-04a **done** (có `uv.lock`), **chưa commit** [ĐÃ XÁC MINH 25/11].

| Đường dẫn | Nội dung |
| --- | --- |
| `README.md` | Mục tiêu, quick start, sơ đồ Mermaid, layout, trạng thái contract, privacy (tiếng Anh) |
| `AGENTS.md` | ≤ 40 dòng cho đồng đội/agent: layout, lệnh, 8 quy tắc (không trỏ tới HARNESS) |
| `pyproject.toml` | Project `fraudweave` (hatchling, version lấy từ `__init__`), deps fastapi + uvicorn, dev: pytest, ruff, httpx2; cấu hình ruff/pytest |
| `.python-version` | `3.11` cho uv |
| `uv.lock` | Lockfile do `uv sync` sinh (commit; không bị `.gitignore` chặn) |
| `.gitignore` | Theo git-workflow §3 (secrets, Python/uv, caches, tmp, editor) |
| `.env.example` | Placeholder: APP_MODE, FW_HMAC_KEY, DATABASE_URL, POSTGRES_PASSWORD, API_KEYS |
| `docker-compose.yml` | Postgres 16 local cho dev/test; bắt buộc có `POSTGRES_PASSWORD` trong `.env` |
| `src/fraudweave/__init__.py` | `__version__` (nguồn version duy nhất) |
| `src/fraudweave/api/main.py` | `create_app()` + mount router dưới `/v1` |
| `src/fraudweave/api/routers/health.py` | `GET /v1/health` → `{status, app_version}` |
| `src/fraudweave/common/contracts.py` | Pydantic contracts §4.2 + transfer-risk (K03) + evidence (K05); marker K06/K07 (T-05) |
| `src/fraudweave/api/errors.py` | Error envelope §7: `ApiError`, `ErrorCode`, handlers (T-05) |
| `src/fraudweave/api/openapi.py` | Sinh OpenAPI kèm mọi contract model (T-05) |
| `scripts/export_openapi.py` | Ghi `contracts/openapi.json`; `--check` để kiểm drift (T-05) |
| `contracts/openapi.json` | OpenAPI sinh tự động (T-05) |
| `tests/contract/test_{models,errors,openapi,privacy}.py` | Contract tests T-05 (privacy: STK thô không lọt vào log/response/error) |
| `src/fraudweave/common/golden.py` | Model `GoldenCase`/`GoldenExpected` + `load_golden_cases()` cho golden fixtures (T-06) |
| `contracts/fixtures/transfer/*.json` | 12 golden case G1, G2, R1–R4, A1–A4, Y1, E1 (A4 ngoài contract §8, proposed D-040); có `*_forbidden`, `evidence_verified_min` (synthetic, STK dải `999000…`; A3 dùng A′ `9990000000010` để không trùng request G1), expected dạng "phải chứa" (T-06) |
| `contracts/CONTRACT_STATUS.md` | Bản sao gọn K01–K12 (tiếng Anh) cho đồng đội + ghi chú K03 cho consumer (T-06) |
| `tests/contract/test_fixtures_schema.py` | Fixture parse được, khớp bảng golden, bất biến "Xanh cần claim", bản sao status khớp marker (T-06) |
| `.gitattributes` | `contracts/**/*.json` luôn LF (test so byte của openapi.json) |
| `tests/unit/test_health.py` | 2 test cho health |
| `.gitkeep` | Thư mục chưa dùng: `contracts/fixtures/{normalize,qr}`, `src/fraudweave/{common/vietqr,safetransfer,evidence,registry,storage,integrations}`, `migrations`, `data/seed`, `web/safetransfer-check`, `tests/golden`, `docs` |

Chưa có (theo PLAN): `.worktreeinclude`, `scripts/new_worktree.ps1`, `check_prepush.ps1` (T-04b), `docs/vietqr-spec-notes.md` (T-07a).

## 3. SANDBOX — `Test-for-agent\`

Hiện trống. Mỗi thử nghiệm là một thư mục `YYYY-MM-DD-<slug>/` có README 3 dòng; ghi một dòng vào đây khi tạo.

| Đường dẫn | Nội dung |
| --- | --- |
| — | — |

## 4. WORKTREES — `worktrees\`

Worktree của REPO cho phiên song song (`context/git-workflow.md` §5). Thư mục chưa tồn tại; script tạo khi cần. Mở hoặc đóng worktree thì sửa bảng này.

| Nhánh | Đường dẫn | Vùng module | Task | Agent | Giữ khoá contract? |
| --- | --- | --- | --- | --- | --- |
| — | — | — | — | — | — |
