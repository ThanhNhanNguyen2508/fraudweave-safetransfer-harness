# PLAN — SafeTransfer (TV3)

> Bám gate **G1 29/11 · G2 5/12 · G3 9/12** (brief §10). Mỗi task ≤ ½ ngày. Làm task bằng skill `implement-task`.
> Trạng thái: `todo` · `doing` · `done` · `manual` (TV3 tự làm, agent chỉ hỗ trợ) · `blocked-pending` (chờ contract, không làm được kể cả với stub).
> Task chạm contract PENDING vẫn làm được: ghi **"stub trước"** + ID Kxx. Đường dẫn file tương đối với REPO; `src/fw/` viết tắt cho `src/fraudweave/`.
> **Đường găng G1:** T-04a → T-05 → T-06 → (T-07, T-08, T-09, T-10, T-14 → T-15, T-16, T-17, T-18) → T-19.
> "DB thật" = **Postgres local qua `docker-compose.yml`**. Supabase chỉ dùng khi demo (K12) (D-022).
> Review 25/11 đã áp dụng: D-018 … D-028.
> **Cách chạy (D-028):** 26/11 chạy **một phiên tuần tự** theo đường găng G1. Nếu **tối 27/11 T-17 chưa xong** → làm T-04b, rồi chuyển sang chạy song song bằng worktree.
> **Git (D-017):** TV3 tự tạo nhánh, commit, push; agent chỉ chạy `git status/diff/log` và gợi ý commit message.

## Giai đoạn 0 — Harness + khung REPO + contract cho đồng đội (25–27/11)

| ID | Mô tả | File trong REPO | Test chứng minh xong | Phụ thuộc | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| T-01 | Harness tầng 1: CLAUDE, AGENTS, MAP, context/, deliverables/ | — (HARNESS) | Có đủ file, CLAUDE.md ≤ 120 dòng | — | done |
| T-02 | Harness tầng 2: SPEC, PLAN, CONTRACT_STATUS, rules, skills, agents, settings | — (HARNESS) | Kiểm tra tự thân: cây HARNESS không có code sản phẩm | T-01 | done |
| T-03 | Harness tầng 3: **TV3 tự** tạo private repo, đặt email noreply local, commit và push HARNESS (agent không chạy git add/commit/push/gh). Tài liệu git-workflow, codex-inbox, `close-session` đã xong | — (HARNESS) | `git log` có commit đầu; remote private có đủ file | T-02 | manual |
| T-04a | Khung REPO: `pyproject.toml` (uv, ruff, pytest), `.gitignore`, `.env.example`, `README.md`, `AGENTS.md` ngắn, cây thư mục, **`docker-compose.yml` chạy Postgres local cho test**, app FastAPI + `GET /v1/health` | `pyproject.toml`, `.gitignore`, `.env.example`, `docker-compose.yml`, `src/fw/api/main.py`, `src/fw/api/routers/health.py` | `uv sync` sinh `uv.lock` (không bị ignore); `uv run pytest` (`tests/unit/test_health.py`) + `uv run ruff check` pass; `uv run uvicorn` trả `/v1/health` (D-029: bước Docker chuyển sang T-14) | — | **done** 25/11: uv 0.13.10, 2 pass (`-W error`), ruff check + format sạch, `/v1/health` 200 |
| T-04b | Công cụ chạy song song: `.worktreeinclude`, `scripts/new_worktree.ps1` (xử lý cả nhánh đã tồn tại), `scripts/check_prepush.ps1`. **Ngoài đường găng G1**; chỉ làm khi kích hoạt điều kiện D-028 (hoặc trước phiên song song đầu tiên), đây cũng là bước kiểm chứng D-014. TV3 tự chạy script tạo worktree | `.worktreeinclude`, `scripts/new_worktree.ps1`, `scripts/check_prepush.ps1` | Tạo rồi xoá được 1 worktree thật ở `Implements\worktrees\` (có chép `.env`); `check_prepush.ps1` bắt được secret giả cài thử | T-04a | todo |
| T-05 | Pydantic contracts theo contract §4.2 + error envelope + script export OpenAPI (K03, K05 DRAFT; phần K06/K07 stub trước) | `src/fw/common/contracts.py`, `src/fw/api/errors.py`, `scripts/export_openapi.py`, `contracts/openapi.json` | `tests/contract/test_models.py` (validate ví dụ trong contract); `openapi.json` sinh lại không đổi | T-04a | done |
| T-06 | 12 file golden fixture (11 ca contract + A4, D-040) (request + expected) + `contracts/CONTRACT_STATUS.md` bản sao | `contracts/fixtures/transfer/*.json`, `contracts/CONTRACT_STATUS.md` | `tests/contract/test_fixtures_schema.py`: mọi fixture parse được bằng models | T-05 | done |
| T-06b | *(Tuỳ chọn)* `mock_server.py`: FastAPI trả response từ fixture theo request, để TV2 code UI trước khi có API thật | `scripts/mock_server.py` | `tests/contract/test_mock_server.py`: mỗi golden request trả đúng expected | T-06 | todo |
| T-06c | **Gửi contract + fixtures cho TV1/TV2** (khoảng 26–27/11), dùng skill `handoff-note`: models, OpenAPI, 11 golden, câu hỏi K06–K10 **Kèm (D-035…D-040, proposed):** bảng K04 v2, C3/C6/C7–C9, `contradictions[]` effective, fixture forbidden + golden A4, D-032/D-033. | — (HARNESS `deliverables/handoff/`) | 2 ghi chú bàn giao (TV1, TV2) có trong `deliverables/handoff/`; TV3 xác nhận đã gửi | T-06 (T-06b nếu có) | done |
| T-06d | **Họp chốt K06–K10 trước 30/11** với TV1/TV2; agent soạn agenda từ `CONTRACT_STATUS.md`, TV3 chủ trì, sau họp cập nhật trạng thái K và `decisions.md` | — (HARNESS) | Biên bản trong `deliverables/`; mỗi K06–K10 có trạng thái mới hoặc lý do còn PENDING | T-06c | manual |

## Giai đoạn 1 — fw_common + DB + transfer-risk → Gate G1 (26–29/11)

| ID | Mô tả | File trong REPO | Test chứng minh xong | Phụ thuộc | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| T-07a | **Đối chiếu spec VietQR/NAPAS và nguồn BIN**: ghi `source_url` cho từng điểm (tag 38 GUID, service code, tag 54/62, CRC); chỉ dùng mẫu QR **công khai** (mẫu nào chứa STK thật thì chỉ đối chiếu cấu trúc trong SANDBOX, không đưa vào fixtures); ghi nguồn cho danh sách BIN. Bắt buộc xong trước khi K02 lên FROZEN | `docs/vietqr-spec-notes.md` | Mọi quy tắc parse trong T-10 trỏ được tới một dòng có `source_url`; mọi BIN trong `bank_aliases.json` có nguồn | — | done |
| T-07 | Bảng ngân hàng: BIN ↔ short name ↔ aliases, kèm nguồn từ T-07a (K01); **chỉ 10 BIN đã có nguồn (D-042)** | `src/fw/common/banks.py`, `src/fw/common/data/bank_aliases.json` | `tests/unit/test_banks.py`: "VCB", "Vietcombank", "970436" → cùng BIN; tên lạ → `unknown_bank` | T-04a (nguồn: T-07a) | done |
| T-08 | Normalise STK (bỏ space/./-, 6–19 số, giữ số 0 đầu) + `recipient_key` HMAC + `account_masked` (K01) | `src/fw/common/normalize.py`, `src/fw/common/hashing.py` | `tests/unit/test_normalize.py`, `test_hashing.py` (key ổn định, không log STK thô) | T-07 | done |
| T-09 | CRC-16/CCITT-FALSE + parser/encoder TLV EMVCo (K02) | `src/fw/common/vietqr/crc.py`, `tlv.py` | `tests/unit/test_crc.py` (check value chuẩn của "123456789" là `0x29B1`), `test_tlv.py` | T-04a | done |
| T-10 | Parse payload VietQR: tag 38 (BIN + STK, service type), 54, 62, kiểm CRC → `NormalizedRecipient` (K02). **Phạm vi D-042/D-043:** chỉ `QRIBFTTA`, 10 BIN, Length theo ký tự, tag 54 chỉ chữ số; trích ID `VQ-xx` / `VQ-Qx` của `docs/vietqr-spec-notes.md` trong test. **Lưu ý từ review T-09:** `crc_matches` chỉ xét 8 ký tự cuối, nên T-10 phải kiểm bằng `parse_tlv` rằng tag `63` là object cuối; bắt cả `TlvError` lẫn `UnknownBankError` → `invalid_qr` / `unknown_bank` | `src/fw/common/vietqr/parse.py` | `tests/unit/test_vietqr_parse.py` + `contracts/fixtures/qr/` (hợp lệ, sai CRC → `invalid_qr`) | T-08, T-09 (nên có T-07a) | done |
| T-13 | Fixture normalise: 1 người nhận viết 6 cách → 1 `recipient_key` (VietQR viết tay trong fixture; ca "QR có amount" dùng payload tĩnh) | `contracts/fixtures/normalize/*.json` | `tests/contract/test_normalize_fixtures.py` | T-10 | **done** 28/11: TV3 nghiệm thu PASS; 726 pass + 159 skip (159 test cần Postgres, chưa kết nối trong phiên agent), ruff sạch; review Opus PASS |
| T-14 | DB: engine/session, models SQLAlchemy (registry, group_trust, evidence, audit_log), Alembic migration đầu tiên, chạy trên Postgres local (docker-compose) | `src/fw/storage/db.py`, `models.py`, `migrations/` | `docker compose up -d db` + `pg_isready` OK (D-029); `tests/unit/test_migrations.py`: upgrade/downgrade trên Postgres local | T-04a | done |
| T-15 | Seed synthetic có cờ cho **cả 12 golden** (G1, G2, R1–R4, A1–A4, Y1; mở rộng từ G1/R1/Y1 vì T-16/T-19 cần org, group, never_asks, evidence unverified) + script seed idempotent | `data/seed/*.json`, `scripts/seed_db.py` | `tests/unit/test_seed.py`: chạy 2 lần không nhân đôi; mọi bản ghi có cờ synthetic; alias không trùng nhau sau chuẩn hoá | T-14, T-08 | done |
| T-16 | Repository đọc: registry, group trust, evidence (lọc hết hạn, lọc simulation theo `mode`) | `src/fw/registry/repository.py`, `src/fw/evidence/repository.py` | `tests/unit/test_repositories.py` | T-14 | done |
| T-17 | Policy: decision table thuần (pure function) + `POLICY_VERSION` + reason codes/`message_vi` (K04) | `src/fw/safetransfer/policy.py`, `src/fw/common/reasons.py` | `tests/unit/test_policy.py`: table-driven, phủ **7 luật bảng v2 (D-035)** + ghép cặp claim–tài khoản và miễn C3 khi khớp official (D-048) + `POLICY_VERSION = st-policy-0.2` (D-041) + các bất biến SPEC §4 (Xanh cần claim, C6 thắng khớp official, registry_conflict không che hard code, amount_band không đổi verdict, A2/A4 không bắn C3, A3 không liệt kê C5); golden runner kiểm `forbidden` | T-05 | done |
| T-18 | Name inquiry mock + claim providers: `user_declared` thật, **resolve `claimed_org_raw` → `org_id` bằng khớp alias chính xác sau chuẩn hoá (chữ thường, bỏ dấu), không khớp mờ; không khớp → `org_not_in_registry`** (SPEC §4.1); `screening_id` **stub trước** (K07) | `src/fw/safetransfer/name_inquiry_mock.py`, `claim_providers.py`, `src/fw/common/text.py` | `tests/unit/test_claim_providers.py`: "EVN-demo", "evn-demo", "  EVN-DEMO " khớp; tên có dấu khớp với alias không dấu (ví dụ "Điện lực Demo" ↔ alias "dien luc demo"); "EVN-dem" (thiếu chữ) và lỗi gõ không khớp → `org_not_in_registry`; `test_name_inquiry_mock.py` | T-05, T-16 | done |
| T-19 | Service + `POST /v1/transfer-risk` + auth API key + audit log (không ghi định danh thô) (K03) | `src/fw/safetransfer/service.py`, `src/fw/api/routers/transfer.py`, `src/fw/api/deps.py` | `tests/golden/test_transfer_golden.py` pass **golden G1, R1, Y1, E1 trên Postgres local** → **Gate G1** | T-06, T-10, T-15, T-16, T-17, T-18 | done |

## Giai đoạn 2 — Đủ tính năng độc lập (29/11–1/12, xong trước 1/12)

| ID | Mô tả | File trong REPO | Test chứng minh xong | Phụ thuộc | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| T-11 | Encode VietQR + sinh ảnh QR demo (K02). **Ngoài G1, xong trước 1/12** | `src/fw/common/vietqr/encode.py`, `scripts/gen_demo_qr.py` | `tests/unit/test_vietqr_roundtrip.py`: `parse(encode(x)) == x` | T-10 | **done** 28/11: TV3 nghiệm thu PASS; 839 pass + 159 skip (159 test cần Postgres, chưa kết nối trong phiên agent), ruff sạch, 5/5 mutant bị bắt; review Opus PASS |
| T-12 | Decode ảnh → payload (opencv `QRCodeDetector`) + `POST /v1/qr/decode` (K02 DRAFT-TV3, D-053: định dạng upload mặc định là multipart một trường ảnh, TV2 xác nhận; **không đặt marker PENDING**). **Ngoài G1, xong trước 1/12** | `src/fw/common/vietqr/decode_image.py`, `src/fw/api/routers/qr.py` | `tests/unit/test_decode_image.py` dùng ảnh sinh từ T-11; `tests/contract/test_qr_endpoint.py` | T-11 | **done** 28/11: TV3 nghiệm thu; phương án B + D-055 (1 decode mỗi client, timeout → 503); review Opus lần 3 không blocker; 1008 pass + 159 skip với `-W error` trong phiên agent (không DB), TV3 báo đã chạy xong PostgreSQL regression (số liệu chi tiết chưa được ghi lại) |
| T-20 | Seed + fixture cho 8 golden còn lại (G2, R2, R3, R4, A1, A2, A3, A4) | `data/seed/*.json`, `contracts/fixtures/transfer/` | `tests/golden/test_transfer_golden.py` pass **đủ 11 + A4** | T-19 | **done** 28/11: TV3 nghiệm thu; audit đạt sẵn theo D-050 (golden 26/26 trên Postgres do TV3 chạy), chỉ thêm `tests/golden/test_golden_preconditions.py` (DB-less, 15 test); review Opus không blocker |
| T-21 | `POST /v1/evidence`: validate, dedup `(key, scam_type)`, status do server đặt, TTL, merge chỉ nâng status (K05; trường từ Guardian/mod/Báo cáo **stub trước** K06) | `src/fw/evidence/ingestion.py`, `src/fw/api/routers/evidence.py` | `tests/unit/test_ingestion.py`, `tests/contract/test_evidence_endpoint.py` | T-16 | **done** (28/11, TV3 nghiệm thu PASS; nhánh `tv3/evidence-ingestion`, TV3 tự commit): PostgreSQL local: full regression 1316 pass (chạy **trước** khi thêm 3 test cuối), evidence ingestion + endpoint 128 pass, golden 41 pass, 3 test bổ sung cuối (report độ tin thấp không đổi verified, lost-race ×2) pass trong lần chạy evidence; ruff, format, OpenAPI, `uv lock --check` sạch. D-057, D-058 duyệt trong phạm vi MVP; K05 DRAFT-TV3, K06 PENDING. Đường dẫn thật `src/fraudweave/...` |
| T-22 | `PUT /v1/groups/{guild_hash}/trust` (payload `/setup` **stub trước** K09, dùng contract §5.4) + `GET /v1/registry/orgs?q=` | `src/fw/api/routers/groups.py`, `registry.py` | `tests/contract/test_groups_endpoint.py`, `test_registry_endpoint.py`; chỉ lưu `recipient_key` + masked | T-16 | todo |
| T-23 | Trang SafeTransfer Check: nhập STK / tải ảnh QR + "tự xưng là" + **ô chọn demo group (seed "Lop 5A", cho golden G2, R3)** + role → verdict; xử lý trạng thái "Không kiểm tra được" | `web/safetransfer-check/index.html`, mount static trong `main.py` | `tests/unit/test_web_static.py` (GET trả 200, có ô chọn group) + kiểm tra tay G2 (Xanh) và R3 (Đỏ) qua form, có ghi lại | T-19, T-12, T-20 | todo |
| T-24 | Eval set ≥ 150 ca có nhãn, **nhãn viết độc lập với policy** (viết expected từ mô tả kịch bản trước khi chạy `policy.py`, runner không bao giờ ghi nhãn); **nhiều ca đầu vào nhiễu** (khoảng trắng, dấu chấm, alias, thiếu dấu, gõ sai, sai ngân hàng, QR hỏng); runner tính accuracy + số Xanh sai; mọi báo cáo ghi **"synthetic, không phải hiệu năng thực tế"**; **freeze 1/12** | `contracts/fixtures/eval/`, `scripts/run_eval.py` | `tests/golden/test_eval_set.py`: ≥ 95% đúng, 0 Xanh sai; kiểm tra tỉ lệ ca nhiễu đạt ngưỡng SPEC §3; output runner có dòng cảnh báo synthetic | T-20, T-21 | todo |
| T-25 | Rate limit 60 req/phút/key + health trả version (policy, contract) | `src/fw/api/deps.py`, `routers/health.py` | `tests/unit/test_rate_limit.py` | T-19 | **done** 28/11: commit `48999ae`, 866 pass trên Postgres (TV3 chạy) |

## Giai đoạn 3 — Tích hợp → Gate G2 (1–5/12)

| ID | Mô tả | File trong REPO | Test chứng minh xong | Phụ thuộc | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| T-26 | Screening cache in-memory TTL 30 phút + resolve `screening_id`, `screening_expired` (**stub trước** K07, K08) | `src/fw/integrations/guardian_adapter.py`, `src/fw/safetransfer/claim_providers.py` | `tests/unit/test_screening_cache.py` (hết hạn, không tồn tại, thắng `claim_context`) | T-18 | todo |
| T-27 | Adapter evidence từ Guardian / mod confirm / nút Báo cáo (**stub trước** K06) | `src/fw/integrations/guardian_adapter.py` | `tests/contract/test_guardian_evidence_adapter.py` | T-21 | todo |
| T-28 | Bàn giao lần 2 cho TV1/TV2: API thật + `fw_common` đã chạy được (TV1 import và chạy normalize fixtures trong CI; TV2 chuyển từ mock sang API thật); cập nhật theo kết quả T-06d | — (HARNESS `deliverables/handoff/`) | TV1/TV2 xác nhận đã nhận và chạy được | T-20, T-22, T-06d | todo |
| T-29 | Nhận `screening_id` từ nút "Chuyển khoản" trên web Check (**stub trước** K10) | `web/safetransfer-check/index.html` | `tests/unit/test_web_static.py` mở rộng | T-23, T-26 | todo |
| T-30 | E2E: Discord message → cảnh báo → "Chuyển khoản" → verdict Đỏ, dùng adapter thật thay stub | `tests/contract/test_e2e_guardian_flow.py`, `docs/` | Test E2E pass → **Gate G2** | T-26–T-29, K06–K10 | blocked-pending |
| T-31 | Chốt stack chung (một app hay hai process), mount router `/v1/screen` của TV1 | `src/fw/api/main.py` | App khởi động với router TV1 | K08, K11 | blocked-pending |

## Giai đoạn 4 — Hardening → Gate G3 (5–9/12)

| ID | Mô tả | File trong REPO | Test chứng minh xong | Phụ thuộc | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| T-32 | Benchmark latency `transfer-risk` / `evidence` trên máy demo | `scripts/bench_latency.py` | Báo cáo p95 ≤ 150 ms (transfer-risk) trong `deliverables/` | T-20 | todo |
| T-33 | Báo cáo eval cuối (≥ 95% đúng, 0 Xanh sai, phân tích ca sai), tiêu đề ghi rõ "synthetic, không phải hiệu năng thực tế" | `scripts/run_eval.py` | File báo cáo trong `deliverables/` | T-24 | todo |
| T-34 | Mở rộng registry lên 50 tổ chức, mỗi dòng có `source_url` + `verified_at` (người ký duyệt: câu hỏi mở) | `data/seed/registry_*.json` | `tests/unit/test_registry_data.py`: mọi dòng thật có `source_url`; alias không trùng sau chuẩn hoá | T-16 | todo |
| T-35 | Deploy demo (K12), Supabase cho môi trường demo | `docs/deploy.md`, scripts liên quan | Health check từ môi trường demo | K12 | blocked-pending |
| T-36 | Demo script + kịch bản dữ liệu | `docs/demo-script.md` | Chạy thử toàn bộ kịch bản một lần, ghi kết quả | T-30, T-32 | todo |

## Phần đã làm sớm (D-050): tái sử dụng và nghiệm thu khi đến đúng task

PLAN của T-20 và T-26 **giữ nguyên**. Chỉ ghi nhận để không làm lại:

- **T-20** (seed + fixture 8 golden còn lại): `data/seed/*.json` và `data/mock/name_inquiry.json` đã phủ cả 12 golden (từ T-15/T-18); `tests/golden/test_transfer_golden.py` đã chạy 12/12 qua HTTP trên Postgres (T-19). Khi đến T-20: rà lại, bổ sung nếu thiếu, rồi nghiệm thu "đủ 11 + A4".
- **T-26** (screening cache + `screening_id`): `InMemoryScreeningCache` (TTL 30 phút, marker K07) và `resolve_claim` (screening_id thắng, lạ/hết hạn → `screening_expired`) đã có ở T-18, kèm test. Khi đến T-26: nghiệm thu phần này, làm adapter phía TV1 (K07/K08) và đổi stub sang cache thật.
- **T-32**: `latency_ms` hiện đo trước khi ghi audit; mỗi request ~8 lượt DB, cần đo nếu Supabase đặt xa (K12).
- **Checklist live (D-049, D-050):** không nạp seed vào live; Name Inquiry Mock không phải nguồn xác minh production (`context/git-workflow.md` §4b).

## Việc dọn riêng (ngoài task, không trộn vào commit của task)

Ghi 28/11 khi nghiệm thu T-13. Mỗi việc một commit nhỏ riêng trong REPO, khi TV3 cho làm:

- `contracts/CONTRACT_STATUS.md` (bản sao REPO) dòng K01 còn ghi "planned `common/normalize.py`, `hashing.py`", đã cũ từ T-08. Sửa cho khớp bản HARNESS; test `test_fixtures_schema.py` chỉ so marker PENDING nên không tự bắt lỗi này.
- `tests/golden/.gitkeep` còn sót dù thư mục đã có `test_transfer_golden.py`; xoá cho đúng `coding-standards.md` (thư mục chưa dùng mới cần `.gitkeep`).

## Ghi chú liên task (D-054, 28/11)

- **T-23 / K12:** trang SafeTransfer Check **không được hardcode API key dịch vụ vào frontend khi deploy công khai**. Đây là câu hỏi mở của T-23/K12 (xem `CONTRACT_STATUS.md`); T-12 không làm BFF.
- **T-21 (`POST /v1/evidence`):** phải dùng `enforce_rate_limit` (contract §7); decode ảnh dùng bucket riêng `enforce_qr_rate_limit`.
- **T-32 (benchmark):** đo thêm `qr/decode` (nguội 0,84 đến 1,23 s, ấm 0,04 đến 0,10 s trên Windows) và `transfer-risk` khi decode bận; p95 ≤ 150 ms vẫn chỉ áp cho `transfer-risk`.
