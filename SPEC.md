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

## 4. Decision table (DRAFT-TV3, K04 — first match wins) — bảng MVP v2 (D-035, **proposed, chờ nhóm duyệt**)

> Thay thế thứ tự ở contract §6 **và brief §7** (cả hai đặt `registry_conflict` ở luật 1). Ghi vào handoff T-06c cho TV1/TV2; chưa freeze.

Đầu vào: `match_official`, `match_declared`, `E_verified`, `E_unverified`, hard contradictions C2/C3/C5/C6, `org_not_in_registry`. **Không** dùng soft signals C7–C9 và pressure words trong `transfer_memo` (D-039).

| # | Điều kiện | Verdict | Action | Reason codes / contradictions |
| --- | --- | --- | --- | --- |
| 0 | Không normalise được người nhận | HTTP 422 | Nhập lại | `invalid_recipient`, `unknown_bank`, `invalid_qr` |
| 1 | Có hard contradiction **C2 / C3 / C6** | red | `cancel` | C2/C3/C6 đã bắn (+ C5 nếu `E_verified ≥ 1`) |
| 2 | (`match_official` or `match_declared`) and `E_verified ≥ 1` | amber | `verify_by_call` | `registry_conflict` (đưa vào review); **không** liệt kê C5 (D-037) |
| 3 | `E_verified ≥ 1` (C5) | red | `cancel` | C5 |
| 4 | `match_official` or `match_declared` | green | `proceed` | `matches_official_account` / `matches_declared_account` (+ `unverified_reports_on_official` nếu `E_unverified ≥ 1`) |
| 5 | `E_unverified ≥ 1`, hoặc org được claim không có trong registry | amber | `verify_by_call` | `unverified_reports`, `org_not_in_registry` |
| 6 | Còn lại | grey | `verify_by_call` | `no_data` |

Định nghĩa hard contradiction (MVP):
- **C2** `payee_not_official_account`: claim resolve được tới org trong registry (hoặc group có `payment_policy = declared_accounts_only` và role teacher / treasurer / admin), và không khớp official/declared.
- **C3** `personal_holder_for_org_claim` (D-036): `holder_type = personal` **và** (claim resolve được tới org trong registry, **hoặc** `claimed_role` ∈ org_staff / bank_staff / police). Org **không** resolve được tự nó không bắn C3; `holder_type = unknown` không bao giờ bắn C3.
- **C5** `confirmed_indicator`: `E_verified ≥ 1`.
- **C6** `org_never_asks_transfer` (D-038): claim resolve được tới org có `never_asks` chứa token **`transfer`**. MVP chỉ hỗ trợ `transfer`; `personal_transfer` và token khác bị bỏ qua. **Limitation:** registry chưa có trường phạm vi áp dụng (role, kênh, loại giao dịch), nên `transfer` áp cho mọi yêu cầu chuyển khoản nhân danh org đó, kể cả vào tài khoản official. Chỉ đặt `transfer` cho dữ liệu synthetic hoặc chính sách có `source_url`.

Ngữ nghĩa `contradictions[]` (D-037):
- **Detected:** mọi bằng chứng tìm thấy luôn được trả trong `evidence_summary` / `evidence`.
- **Effective:** `contradictions[]` chỉ chứa mâu thuẫn hiệu lực. Verdict **red** → mọi hard code đang đúng (C2/C3/C5/C6). Verdict **amber / grey / green** → không có hard code nào. Vì vậy ở luật 2, verified evidence vẫn hiện trong `evidence_summary.verified` nhưng C5 không nằm trong `contradictions[]`.

Bất biến phải có test (T-17):
- **Xanh cần claim hợp lệ.** Không có claim org/group resolve được thì không bao giờ Xanh; vẫn có thể Đỏ (C5) hoặc Vàng.
- **Không bao giờ dùng chữ "an toàn"** cho Xanh. Copy Xám: "Chưa có dữ liệu về tài khoản này — điều này không có nghĩa là an toàn."
- **Báo cáo chưa xác minh không bao giờ đưa lên Đỏ**, và không gỡ được Xanh của tài khoản official/declared.
- **Khớp official không thắng C6 hợp lệ**; **registry_conflict không che hard contradiction độc lập** (luật 1 đứng trước luật 2).
- Simulation evidence chỉ tính khi `mode=demo` (response có `demo_data_used: true`).
- `amount_band` chỉ đổi lời khuyên, không đổi verdict (câu hỏi mở K04).
- Lỗi 5xx/timeout **không bao giờ** thành Xanh.
- `screening_id` thắng `claim_context` nếu gửi cả hai; `screening_id` hết hạn/không tồn tại → bỏ qua + `screening_expired`.
- Mọi response có `policy_version`. Bảng v2: T-17 dùng `st-policy-0.2` (D-041).
- **C7–C9 không ảnh hưởng verdict trong MVP** (D-039). Mã C7–C9 và `ClaimContext.soft_signals` vẫn giữ trong contract ở dạng reserved. **Chờ TV1/TV2 xong rồi mới chốt có implement thêm C7–C9 không**; TV3 sẽ tự nhắc lại khi cần.

### 4.1 Resolve claim tổ chức (D-020)

- Có `claimed_org_id` và id đó tồn tại trong registry → dùng luôn.
- Có `claimed_org_raw` → chuẩn hoá: chữ thường, bỏ dấu tiếng Việt (kể cả `đ` → `d`), cắt khoảng trắng đầu/cuối và gộp khoảng trắng liên tiếp (D-027). Sau đó **khớp chính xác** với `name` hoặc `aliases` (đã chuẩn hoá cùng cách) của registry.
- **Không khớp mờ** (không fuzzy, không prefix, không khoảng cách Levenshtein). Không khớp → claim có tổ chức nhưng không resolve được → reason `org_not_in_registry` (luật 5 bảng v2, Vàng).
- Bất biến dữ liệu: sau chuẩn hoá, không có alias nào trùng giữa hai tổ chức (có test trên seed).

## 5. Golden cases (contract §8, toàn bộ dữ liệu synthetic)

Mỗi case = 1 file JSON trong `REPO/contracts/fixtures/transfer/` (request + expected verdict + expected reason codes). CI phải pass đủ 11 ca của contract **+ A4** (D-040, proposed); TV2 dùng chính các file này làm mock.

So khớp (D-040): verdict / action / HTTP status khớp **chính xác**; C-codes và reason codes là **tập con phải chứa**; thêm **forbidden** để chặn response tự mâu thuẫn (ca không đỏ cấm mọi hard code C2/C3/C5/C6). Dải STK synthetic `999000…`.

| ID | Seed data | Request | Expected |
| --- | --- | --- | --- |
| G1 | Org "EVN-demo" có official account A | A, claim EVN-demo | green, `matches_official_account` |
| G2 | Group "Lop 5A" khai fund account B, policy `declared_accounts_only` | B, claim role treasurer + guild | green, `matches_declared_account` |
| R1 | Account C có 1 evidence verified | C, không claim | red, C5 |
| R2 | Org "VCB-demo" trong registry (không có `never_asks`); account D là cá nhân | D, claim bank_staff của VCB-demo | red, C2 + C3 |
| R3 | Group "Lop 5A" như G2 | account E (khác B), claim teacher + guild | red, C2 |
| R4 | Org "Cong an-demo", `never_asks = ["transfer"]` (D-038) | account I không official, claim police | red, C6 (C2 cũng bắn) |
| A1 | Account F có 1 report unverified | F, không claim | amber, `unverified_reports` |
| A2 | Không dữ liệu; name inquiry G → `unknown` | account G, claim "Quy ABC" (không có trong registry) | amber, `org_not_in_registry`; cấm C3 |
| A4 | Như A2 nhưng name inquiry J → `personal` (D-036, **thêm ngoài contract §8, proposed**) | account J, claim "Quy ABC" | amber, `org_not_in_registry`; cấm C3 |
| A3 | EVN-demo như G1; official account **A′** (không phải A, để G1 vẫn đúng trên cùng seed) có 1 evidence verified | A′, claim EVN-demo | amber, `registry_conflict`; cấm C5; `evidence_summary.verified ≥ 1` (D-037) |
| Y1 | Không dữ liệu | account H, không claim | grey, `no_data` |
| E1 | — | account "12AB" | 422, `invalid_recipient` |

Fixture bổ sung: `contracts/fixtures/normalize/` (cùng 1 người nhận viết 6 cách → 1 `recipient_key`, TV1 chạy trong CI của họ) và `contracts/fixtures/qr/` (payload VietQR hợp lệ / sai CRC).

## 6. Định nghĩa "Done" theo gate

| Gate | Ngày | Done khi |
| --- | --- | --- |
| **G1** | 29/11 | `fw_common` có test: normalise + **VietQR parse + CRC**; DB schema + migrations + seed synthetic chạy được; `POST /v1/transfer-risk` pass **golden G1, R1, Y1, E1 trên DB thật = Postgres local (docker-compose)**; `ruff` sạch; `/v1/health` trả `policy_version` |
| Mốc giữa | 1/12 | **VietQR encode + decode ảnh + `POST /v1/qr/decode`**; đủ **11 golden + A4** pass (D-040); evidence ingestion; name inquiry mock; trang SafeTransfer Check chạy được (có ô chọn demo group); **eval set ≥ 150 ca freeze 1/12** |
| **G2** | 5/12 | K06–K10 đã chốt hoặc có adapter thật thay stub; TV2 gọi API thật; luồng Discord message → cảnh báo → "Chuyển khoản" → verdict Đỏ chạy end-to-end |
| **G3** | 9/12 | p95 `transfer-risk` ≤ 150 ms đã đo; báo cáo eval ≥ 95% đúng và 0 Xanh sai; registry 50 tổ chức có `source_url`; demo script hoàn chỉnh |

## 7. Chưa kiểm chứng

- Chi tiết VietQR/EMVCo (tag 38 GUID, service code, tag 62, CRC) và danh sách BIN phải đối chiếu spec NAPAS chính thức, có `source_url`, chỉ dùng mẫu QR công khai, trước khi K02 lên FROZEN (task T-07a, D-021).
- Dữ liệu registry trong contract (EVN, hotline 19009000...) là placeholder.
