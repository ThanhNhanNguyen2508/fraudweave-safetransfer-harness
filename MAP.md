# MAP — bản đồ 3 thư mục

> Cập nhật mỗi khi tạo, xoá hoặc di chuyển file ở **bất kỳ** thư mục nào. Lần cuối: 2026-11-28. Chỉ Claude Code sửa file này; Codex đề xuất qua `deliverables/codex-inbox/`.
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
| `deliverables/handoff/2026-11-27-tv2-transfer-risk-contract.md` | Handoff T-06c cho TV2: models, error envelope, 12 fixture, K04 v2, câu hỏi K02/K03/K04/K06/K09–K11 |
| `deliverables/handoff/2026-11-27-tv2-transfer-risk-endpoint-live.md` | Handoff TV2: endpoint chạy thật (12/12 golden), OpenAPI mới, mã lỗi, `X-Request-Id`, hành vi response, lưu ý name inquiry mock là demo |
| `deliverables/handoff/2026-11-27-tv1-contracts-and-claim-context.md` | Handoff T-06c cho TV1: models dùng chung, C7–C9 reserved, K04 v2, câu hỏi K01/K04/K06–K08/K11 |
| `deliverables/handoff/2026-11-27-tv1-recipient-key-and-normalisation.md` | Handoff TV1 cho T-07/T-08/T-09: hàm `fw_common`, quy tắc K01, bảng map lỗi, 4 vector hồi quy, câu hỏi K01 |
| `deliverables/handoff/2026-11-27-k04-d048-claim-pairing.md` | Handoff TV1 + TV2: K04 D-048 (miễn C3 khi khớp official, ghép cặp claim–tài khoản), bảng ca đổi kết quả, 12 golden không đổi |
| `deliverables/2026-11-28-gate-g1-acceptance.md` | Biên bản nghiệm thu Gate G1 theo SPEC §6: đối chiếu từng tiêu chí, kết quả test (866 pass trên Postgres), contract-audit |
| `deliverables/handoff/2026-11-28-tv2-health-versions-rate-limit.md` | Handoff TV2: health có version, 429 `rate_limited` + `Retry-After`, thứ tự 401→429→400/422, câu hỏi K03/K12 |
| `deliverables/handoff/2026-11-28-tv1-health-versions.md` | Handoff TV1: G1 đạt, health version, chưa có `model_version` (K08), câu hỏi K08/K11 |
| `deliverables/codex-inbox/README.md` | Quy ước và mẫu ghi chú cuối phiên của Codex; file đã nhập chuyển sang `processed/` |
| `sources/00-project-brief.md` | Brief dự án, nguồn sự thật (chỉ đọc) |
| `sources/contract-v0.1.md` | Integration Contract v0.1; §3 đã bị brief §5 thay thế (chỉ đọc) |
| `sources/pipeline-revised.md` | Sơ đồ Mermaid pipeline + ghi chú sau phân công mới (chỉ đọc) |
| `sources/01-harness-prompts.md` | Bộ prompt dựng harness 4 bước (chỉ đọc) |

## 2. REPO — `Source code-SafeTransfer\FraudWaveAI-Momo\`

Git repo sản phẩm, remote `github.com/ThanhNhanNguyen2508/FraudWaveAI-Momo` (tài khoản của TV3). Nhánh hiện tại `tv3/contracts-golden-fixtures` (commit `c5e92c4`, T-05 + T-06 đã commit [ĐÃ XÁC MINH 27/11]; push theo TV3 báo).

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
| `src/fraudweave/api/routers/health.py` | `GET /v1/health` → `{status, app_version, policy_version, contract_version}`, không cần key, không bị rate limit (T-25) |
| `src/fraudweave/api/rate_limit.py` | `RateLimiter` sliding window trong bộ nhớ, 60 req/phút/client (contract §7); `enforce_rate_limit` trong `deps.py` → 429 `rate_limited` + `Retry-After` (T-25) |
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
| `src/fraudweave/common/banks.py` | Bảng ngân hàng (T-07): `Bank`, `BankTable`, `resolve_bank()`, `bank_by_bin()`, `UnknownBankError`; khớp chính xác, chỉ 10 BIN (D-042) |
| `src/fraudweave/common/data/bank_aliases.json` | 10 ngân hàng: BIN, short name, alias, nguồn (T-07) |
| `src/fraudweave/common/normalize.py` | Chuẩn hoá STK (`normalize_account_number`), che số (`mask_account`), `normalize_manual_recipient()` → `NormalizedRecipient`; `InvalidRecipientError` (T-08) |
| `src/fraudweave/common/hashing.py` | `recipient_key()` HMAC-SHA256, `hmac_key_from_env()`, `InvalidHmacKeyError` (T-08) |
| `tests/conftest.py` | Fixture `hmac_key` giả dùng chung cho test |
| `src/fraudweave/safetransfer/policy.py` | Decision table MVP v2 thuần: `PolicyInput` (kiểm nhất quán), `decide()` → `PolicyDecision`, `POLICY_VERSION = st-policy-0.2`, `never_asks_transfer()` (T-17) |
| `src/fraudweave/common/reasons.py` | Reason codes + toàn bộ copy tiếng Việt cho người dùng (`message_vi`, `build_contradiction`) (T-17) |
| `tests/unit/test_{policy,reasons}.py` | Golden inputs viết tay + 7 luật + bất biến chạy vét cạn 7.290 tổ hợp (T-17) |
| `src/fraudweave/common/text.py` | `normalize_text()` chuẩn hoá alias (chữ thường, bỏ dấu, gộp khoảng trắng); T-18 dùng lại |
| `src/fraudweave/common/vietqr/parse.py` | `parse_vietqr(payload, hmac_key)` → `NormalizedRecipient`; `InvalidQrError` (T-10) |
| `contracts/fixtures/qr/*.json` | 17 payload VietQR synthetic (Q01–Q05 hợp lệ, Q10–Q21 lỗi) + kết quả mong đợi (T-10) |
| `tests/unit/test_vietqr_parse.py`, `tests/contract/test_qr_fixtures.py` | Test parser và fixture QR, trích ID `VQ-xx` (T-10) |
| `src/fraudweave/common/vietqr/crc.py`, `tlv.py` | CRC-16/CCITT-FALSE (`append_crc`, `crc_matches`) và TLV EMVCo (`parse_tlv`, `encode_tlv`, `TlvError`) (T-09) |
| `tests/unit/test_{banks,text,crc,tlv}.py` | Test T-07 và T-09, trích ID `VQ-xx` |
| `tests/unit/test_{normalize,hashing}.py` | Test T-08: separators, 6–19 số, số 0 đầu, mask, vector hồi quy HMAC, không log/echo STK |
| `docs/vietqr-spec-notes.md` | T-07a: quy tắc parse VietQR VQ-01…VQ-14 (mỗi dòng có `source_url` + nhãn PRIMARY/SECONDARY/LOCAL), nguồn BIN (NHNN 31/12/2023 + VietQR.IO), cảnh báo alias ngân hàng |
| `tests/unit/test_vietqr_spec_notes.py` | Mỗi dòng VQ có nhãn + https source; BIN 6 số `9704…` không trùng (T-07a) |
| `tests/unit/test_health.py` | Test health: đủ 3 version, không cần key/cấu hình (T-25) |
| `tests/unit/test_rate_limit.py` | Test limiter (cửa sổ trượt, Retry-After, tách client) + 429 trên `transfer-risk` (T-25) |
| `alembic.ini` | Cấu hình Alembic; URL không nằm trong file (T-14) |
| `migrations/env.py`, `script.py.mako`, `versions/0001_initial_schema.py` | Alembic: env đọc `DATABASE_URL` hoặc URL do test truyền; migration đầu tạo 8 bảng + bật RLS (T-14) |
| `src/fraudweave/storage/db.py` | `to_sqlalchemy_url` (psycopg 3), `database_url_from_env`, `make_engine`, `make_session_factory`; không kết nối lúc import (T-14) |
| `src/fraudweave/storage/models.py` | SQLAlchemy: `orgs`, `org_official_accounts`, `groups`, `group_anchors`, `group_fund_accounts`, `evidence`, `evidence_reporters`, `audit_log`; CHECK cho enum, `rk_`, `public_warning` cần `source_url`, simulation (T-14) |
| `tests/unit/test_db.py`, `test_migrations.py` | T-14: URL/model hygiene (không cần DB); migration upgrade/downgrade, drift, RLS, ràng buộc trên Postgres local (cần `FW_TEST_DATABASE_URL` tên `*_test`, không thì skip) |
| `data/seed/{orgs,groups,evidence}.json` | Seed synthetic cho 12 golden (3 org, 1 group, 3 evidence); STK dải `999000…`, mọi bản ghi `synthetic: true`, `source_url` là `example.invalid` (T-15) |
| `src/fraudweave/storage/seed.py` | `load_seed()` (pydantic, `extra=forbid`) + `apply_seed()` idempotent; tính `recipient_key` trong bộ nhớ, chỉ ghi key + số đã che (T-15) |
| `scripts/seed_db.py` | CLI nạp seed (cần `DATABASE_URL`, `FW_HMAC_KEY`); chỉ in số lượng (T-15) |
| `src/fraudweave/registry/repository.py` | `find_org_by_claim` (khớp chính xác sau `normalize_text`, trùng 2 org → None), `get_org`, `get_group_trust`; trả dataclass frozen (T-16) |
| `src/fraudweave/evidence/repository.py` | `lookup_account_evidence`: bỏ retracted/hết hạn; simulation chỉ tính ở `mode=demo` (như verified, `demo_data_used`); top 3 record (T-16) |
| `tests/unit/test_seed.py`, `test_repositories.py` | T-15/T-16 (phần DB cần `FW_TEST_DATABASE_URL`); `tests/conftest.py` có fixture `test_database_url`, `alembic_config`, `clean_db` dùng chung |
| `data/mock/name_inquiry.json` | Name inquiry mock: 5 tài khoản synthetic (STK `999000…`) → `holder_type`; không có trong file = `unknown` (T-18) |
| `src/fraudweave/safetransfer/name_inquiry_mock.py` | `NameInquiryMock.from_file` (pydantic, tính key trong bộ nhớ), `UNKNOWN_HOLDER` (T-18) |
| `src/fraudweave/safetransfer/claim_providers.py` | `resolve_declared_claim`, `resolve_claim` (screening_id thắng, lạ/hết hạn → bỏ qua + `screening_expired`), `InMemoryScreeningCache` stub K07 (T-18) |
| `src/fraudweave/safetransfer/service.py` | `TransferRiskService.assess`: normalise → claim → evidence → name inquiry → `decide` → response + audit; lỗi DB = `ApiError` 503 (T-19) |
| `src/fraudweave/config.py` | `Settings.from_env` (DATABASE_URL, FW_HMAC_KEY, API_KEYS, FW_NAME_INQUIRY_FILE tuỳ chọn), `parse_api_keys`, `client_for_key` so sánh hằng thời gian; secret không hiện trong repr (T-19) |
| `src/fraudweave/api/deps.py`, `routers/transfer.py` | `require_api_key` (`X-API-Key`), `get_service` dựng lười (thiếu cấu hình = 500, không có verdict), `POST /v1/transfer-risk`; `api/main.py` có `create_app(settings, service)` (T-19) |
| `tests/golden/test_transfer_golden.py` | 12 golden qua HTTP trên Postgres thật (Gate G1: G1, R1, Y1, E1) (T-19) |
| `tests/unit/test_{claim_providers,name_inquiry_mock,transfer_service,config}.py`, `tests/contract/test_transfer_endpoint.py` | T-18/T-19: alias chính xác, mock, service + audit + privacy + fail-closed, auth/envelope |
| `.gitkeep` | Thư mục chưa dùng: `contracts/fixtures/normalize`, `src/fraudweave/integrations`, `web/safetransfer-check`, `tests/golden` |

Chưa có (theo PLAN): `.worktreeinclude`, `scripts/new_worktree.ps1`, `check_prepush.ps1` (T-04b).

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
