# SPEC — SafeTransfer (FraudWeave, TV3)

> Nguồn: brief §1–§4, §7; contract v0.1 §4–§8. Trạng thái tổng: **DRAFT-TV3** cho tới khi họp freeze contract.
> Đổi SPEC thì ghi `context/decisions.md` và cập nhật fixtures liên quan.

## 1. Mục đích

Cảnh báo **trước khi chuyển khoản**. Người dùng nhập STK + ngân hàng hoặc tải ảnh/quét VietQR, có thể khai "người nhận tự xưng là…"; hệ thống trả verdict **Đỏ / Vàng / Xám / Xanh** kèm contradictions, evidence và reason codes.

Giá trị riêng so với blacklist của MoMo/SIMO: phát hiện **claim–payee mismatch** (C2, C3), cộng với evidence đã xác minh (C5) và chính sách "không bao giờ yêu cầu" của tổ chức (C6).

## 2. Phạm vi

### Trong phạm vi (TV3 làm)
- `fw_common`: normaliser STK/bank, bank aliases/BIN, `recipient_key` (HMAC), reason codes + `message_vi`, Pydantic contracts.
- VietQR: decode ảnh → payload, parse TLV + CRC, encode QR demo; `POST /v1/qr/decode`.
- Postgres: Org Registry (seed 10 → 50 tổ chức), Group Trust, Evidence (ingestion, dedup, lifecycle, TTL), audit log. Dev và test chạy **Postgres local qua `docker-compose.yml`**; Supabase chỉ dùng cho môi trường demo (K12) (D-022).
- Name Inquiry **mock**.
- SafeTransfer engine: claim providers, policy (decision table), `POST /v1/transfer-risk`.
- Endpoint phụ: `POST /v1/evidence`, `PUT /v1/groups/{guild_hash}/trust`, `GET /v1/registry/orgs`, `GET /v1/health`.
- FastAPI app shell: config, auth API key, error envelope, rate limit, logging.
- Trang web tối giản **SafeTransfer Check** (HTML tĩnh).
- Golden fixtures, eval set ≥ 150 ca, đo latency, tích hợp E2E, demo script.

### Ngoài phạm vi
- Guardian engine (`engine.screen()`, Claim-Check B1–B5, PhoBERT/ONNX) và `POST /v1/screen` → **TV1**.
- Discord bot, tier UI, mod review UI, `/setup` UI, nút "Báo cáo" / "Chuyển khoản", banking simulator đẹp, Discord Activity → **TV2**.
- Name inquiry thật với ngân hàng, dữ liệu khách hàng thật, mô hình ML quyết định verdict.
- Auth ngoài API key (OAuth, user account).

## 3. Mục tiêu đo được

| Chỉ số | Mục tiêu | Đo bằng |
| --- | --- | --- |
| Verdict đúng | ≥ 95% trên eval set ≥ 150 ca có nhãn | Script eval trong REPO, báo cáo trong `deliverables/` |
| Xanh sai | **0** trên golden + eval set | Như trên |
| `POST /v1/transfer-risk` server p95 | ≤ 150 ms | Benchmark trên máy demo |
| End-to-end (web/simulator) | ≤ 300 ms | Đo từ client |
| `POST /v1/evidence` server p95 | ≤ 300 ms | Benchmark |
| Độc lập | Chạy E2E không cần TV1/TV2 | Demo: nhập STK / tải ảnh QR → verdict |

Quy tắc cho eval set (D-023):
- **Nhãn viết độc lập với policy:** expected verdict viết từ mô tả kịch bản và decision table trên giấy, **trước** khi chạy `policy.py`. Runner chỉ đọc nhãn, không bao giờ sinh hoặc sửa nhãn. Nhãn nào bị đổi sau freeze phải có lý do trong `context/decisions.md`.
- **Nhiều ca đầu vào nhiễu:** khoảng trắng, dấu chấm/gạch, alias ngân hàng, viết hoa/thường, thiếu dấu tiếng Việt, gõ sai tên tổ chức, sai ngân hàng, STK sai độ dài, QR hỏng CRC. **≥ 30%** số ca có ít nhất một kiểu nhiễu (D-027).
- **Mọi kết quả** (output runner, báo cáo, slide) luôn ghi: **"synthetic, không phải hiệu năng thực tế"**.

## 4. Decision table (DRAFT-TV3, K04 — first match wins)

Đầu vào: `match_official`, `match_declared`, `E_verified`, `E_unverified`, hard contradictions C2/C3/C5/C6, soft signals C7–C9 (+ pressure words trong `transfer_memo`, rules only).

| # | Điều kiện | Verdict | Action | Reason codes |
| --- | --- | --- | --- | --- |
| 0 | Không normalise được người nhận | HTTP 422 | Nhập lại | `invalid_recipient`, `unknown_bank`, `invalid_qr` |
| 1 | (`match_official` or `match_declared`) and `E_verified ≥ 1` | amber | `verify_by_call` | `registry_conflict` (đưa vào review) |
| 2 | Có hard contradiction C2/C3/C5/C6 | red | `cancel` | các C-code đã kích hoạt |
| 3 | `match_official` or `match_declared` | green | `proceed` | `matches_official_account` / `matches_declared_account` (+ `unverified_reports_on_official` nếu `E_unverified ≥ 1`) |
| 4 | `E_unverified ≥ 1`, hoặc soft signal, hoặc org được claim không có trong registry | amber | `verify_by_call` | `unverified_reports`, C7–C9, `org_not_in_registry` |
| 5 | Còn lại | grey | `verify_by_call` | `no_data` |

Bất biến phải có test:
- **Xanh cần claim.** Không có claim org/group thì tối đa là Xám.
- **Không bao giờ dùng chữ "an toàn"** cho Xanh. Copy Xám: "Chưa có dữ liệu về tài khoản này — điều này không có nghĩa là an toàn."
- **Báo cáo chưa xác minh không bao giờ đưa lên Đỏ**, và không gỡ được Xanh của tài khoản chính thức.
- Simulation evidence chỉ tính khi `mode=demo` (response có `demo_data_used: true`).
- `amount_band` chỉ đổi lời khuyên, không đổi verdict (câu hỏi mở K04).
- Lỗi 5xx/timeout **không bao giờ** thành Xanh.
- `screening_id` thắng `claim_context` nếu gửi cả hai; `screening_id` hết hạn/không tồn tại → bỏ qua + `screening_expired`.
- Mọi response có `policy_version` (hiện `st-policy-0.1`).
- **Xanh thắng tín hiệu mềm C7–C9** theo thứ tự hiện tại (luật 3 đứng trước luật 4). Đây là **câu hỏi mở K04** (D-026): có thể đổi thành "khớp official nhưng có C7–C9 → Vàng".

### 4.1 Resolve claim tổ chức (D-020)

- Có `claimed_org_id` và id đó tồn tại trong registry → dùng luôn.
- Có `claimed_org_raw` → chuẩn hoá: chữ thường, bỏ dấu tiếng Việt (kể cả `đ` → `d`), cắt khoảng trắng đầu/cuối và gộp khoảng trắng liên tiếp (D-027). Sau đó **khớp chính xác** với `name` hoặc `aliases` (đã chuẩn hoá cùng cách) của registry.
- **Không khớp mờ** (không fuzzy, không prefix, không khoảng cách Levenshtein). Không khớp → claim có tổ chức nhưng không resolve được → reason `org_not_in_registry` (luật 4, Vàng).
- Bất biến dữ liệu: sau chuẩn hoá, không có alias nào trùng giữa hai tổ chức (có test trên seed).

## 5. Golden cases (contract §8, toàn bộ dữ liệu synthetic)

Mỗi case = 1 file JSON trong `REPO/contracts/fixtures/transfer/` (request + expected verdict + expected reason codes). CI phải pass đủ 11; TV2 dùng chính các file này làm mock.

| ID | Seed data | Request | Expected |
| --- | --- | --- | --- |
| G1 | Org "EVN-demo" có official account A | A, claim EVN-demo | green, `matches_official_account` |
| G2 | Group "Lop 5A" khai fund account B, policy `declared_accounts_only` | B, claim role treasurer + guild | green, `matches_declared_account` |
| R1 | Account C có 1 evidence verified | C, không claim | red, C5 |
| R2 | Org "VCB-demo" trong registry; account D là cá nhân | D, claim bank_staff của VCB-demo | red, C2 + C3 |
| R3 | Group "Lop 5A" như G2 | account E (khác B), claim teacher + guild | red, C2 |
| R4 | Org "Cong an-demo", `never_asks` = transfer | account bất kỳ, claim police | red, C6 |
| A1 | Account F có 1 report unverified | F, không claim | amber, `unverified_reports` |
| A2 | Không dữ liệu | account G, claim "Quy ABC" (không có trong registry) | amber, `org_not_in_registry` |
| A3 | Account A (như G1) có thêm 1 evidence verified | A, claim EVN-demo | amber, `registry_conflict` |
| Y1 | Không dữ liệu | account H, không claim | grey, `no_data` |
| E1 | — | account "12AB" | 422, `invalid_recipient` |

Fixture bổ sung: `contracts/fixtures/normalize/` (cùng 1 người nhận viết 6 cách → 1 `recipient_key`, TV1 chạy trong CI của họ) và `contracts/fixtures/qr/` (payload VietQR hợp lệ / sai CRC).

## 6. Định nghĩa "Done" theo gate

| Gate | Ngày | Done khi |
| --- | --- | --- |
| **G1** | 29/11 | `fw_common` có test: normalise + **VietQR parse + CRC**; DB schema + migrations + seed synthetic chạy được; `POST /v1/transfer-risk` pass **golden G1, R1, Y1, E1 trên DB thật = Postgres local (docker-compose)**; `ruff` sạch; `/v1/health` trả `policy_version` |
| Mốc giữa | 1/12 | **VietQR encode + decode ảnh + `POST /v1/qr/decode`**; đủ **11 golden** pass; evidence ingestion; name inquiry mock; trang SafeTransfer Check chạy được (có ô chọn demo group); **eval set ≥ 150 ca freeze 1/12** |
| **G2** | 5/12 | K06–K10 đã chốt hoặc có adapter thật thay stub; TV2 gọi API thật; luồng Discord message → cảnh báo → "Chuyển khoản" → verdict Đỏ chạy end-to-end |
| **G3** | 9/12 | p95 `transfer-risk` ≤ 150 ms đã đo; báo cáo eval ≥ 95% đúng và 0 Xanh sai; registry 50 tổ chức có `source_url`; demo script hoàn chỉnh |

## 7. Chưa kiểm chứng

- Chi tiết VietQR/EMVCo (tag 38 GUID, service code, tag 62, CRC) và danh sách BIN phải đối chiếu spec NAPAS chính thức, có `source_url`, chỉ dùng mẫu QR công khai, trước khi K02 lên FROZEN (task T-07a, D-021).
- Dữ liệu registry trong contract (EVN, hotline 19009000...) là placeholder.
