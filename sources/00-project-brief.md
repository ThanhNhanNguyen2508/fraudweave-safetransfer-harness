# FraudWeave — SafeTransfer (TV3) · Project Brief cho Agent Workspace

> File này là **nguồn sự thật** để agent (Claude Code / Codex) dựng harness và làm việc.
> Đặt file vào `Agent Workspace/sources/00-project-brief.md`. Khi có quyết định mới, cập nhật file này hoặc ghi vào `context/`.
> Cập nhật lần cuối: 25/11/2026 · Chủ sở hữu: TV3 (Rykanny)

---

## 1. Bối cảnh

- Cuộc thi: MoMo Tech Contest 2026 (chủ đề Anti-Fraud / Cybersecurity). Gate G1 29/11 · G2 5/12 · G3 9/12 · TechDay 11/12/2026.
- Sản phẩm: **FraudWeave**, MVP chỉ có 2 tính năng:
  1. **Guardian** (TV1 + TV2): bot Discord phát hiện tin nhắn lừa đảo mạo danh theo thời gian thực (Claim-Check B1–B5, tier T0–T3).
  2. **SafeTransfer** (TV3, mình): cảnh báo **trước khi chuyển khoản**. Người dùng nhập STK hoặc quét VietQR, hệ thống trả về verdict **Đỏ / Vàng / Xám / Xanh** kèm bằng chứng và lý do.
- Hai tính năng nối với nhau qua **Shared Fraud Evidence Store** (Supabase Postgres) và **claim context** (người nhận tự xưng là ai).

## 2. Ý tưởng cốt lõi của SafeTransfer (vì sao không chỉ là blacklist)

- MoMo (AI cảnh báo người nhận) và SIMO đã có blacklist ở thời điểm chuyển tiền. Nếu SafeTransfer chỉ tra blacklist thì không có giá trị mới.
- Giá trị của mình là **claim–payee mismatch**: người nhận *tự xưng* là tổ chức hoặc vai trò X, nhưng tài khoản lại không khớp với tài khoản chính thức hoặc đã khai báo của X, hoặc là tài khoản cá nhân.
- Các contradiction SafeTransfer tự tính: **C2** (payee ≠ official/declared account), **C3** (chủ tài khoản cá nhân nhưng claim là tổ chức, qua name inquiry *mock*), **C5** (indicator đã được xác minh), **C6** (tổ chức "không bao giờ yêu cầu" hành động này). Tín hiệu mềm C7–C9 đến từ Guardian (sau khi tích hợp).

## 3. Mục tiêu đo được

- Verdict đúng ≥ 95% trên eval set ≥ 150 ca chuyển khoản có nhãn. **0 verdict Xanh sai.**
- `POST /v1/transfer-risk`: server p95 ≤ 150 ms, end-to-end ≤ 300 ms.
- Chạy được **end-to-end độc lập** (không cần TV1/TV2): nhập STK hoặc tải ảnh QR → verdict + bằng chứng, trên một trang web tối giản và qua API.

## 4. Nguyên tắc làm việc đã chốt

1. **TV3 tự chủ end-to-end SafeTransfer.** Mọi đầu vào bắt buộc của SafeTransfer do TV3 sở hữu, kể cả QR (decode ảnh, parse, encode). TV3 không phải chờ TV1/TV2.
2. **"Làm được nhiều nhất có thể" (build-first, integrate-later).** Phần contract chưa chốt với TV1/TV2 được đánh dấu **`PENDING-NEED-CONFIRM-CONTRACT`**, có stub/adapter để code vẫn chạy. Không chờ contract hoàn hảo.
3. **Contract-first trong nội bộ TV3.** Pydantic models là nguồn sự thật; OpenAPI và fixtures sinh ra từ đó. TV2 và TV1 dùng chính fixtures này làm mock khi tích hợp.
4. **Evidence over assumptions.** Không bịa dữ liệu thật. Mọi dữ liệu demo đều là synthetic và được gắn cờ. Dữ liệu tổ chức thật phải có `source_url`.
5. **Privacy by design.** Không lưu nội dung tin nhắn, STK thô, Discord ID hay số tiền chính xác. Chỉ lưu `recipient_key` (HMAC), số đã che và `amount_band`.
6. **Deterministic policy.** Verdict SafeTransfer đến từ một decision table có thứ tự, không phải điểm ML. Mỗi verdict có reason codes và `policy_version`.

## 5. Phân công (đã điều chỉnh 25/11, thay thế §3 của Contract v0.1)

| Module | Owner | Ghi chú |
| --- | --- | --- |
| VietQR: **decode ảnh → payload**, parse TLV + CRC, **encode** (sinh QR demo/test) | **TV3** | Chuyển từ TV2 sang TV3 vì là đầu vào bắt buộc. Có endpoint `POST /v1/qr/decode` để TV2 dùng lại |
| `fw_common`: normaliser, bank aliases/BIN, `recipient_key`, reason codes, Pydantic contracts | **TV3** | TV1 import, không tự viết lại |
| Org Registry (seed demo 10 tổ chức do TV3 tự curate, mở rộng lên 50 sau) | **TV3** | TV1 có thể bổ sung dữ liệu sau khi tích hợp |
| Group Trust store + API | **TV3** | UI `/setup` trên Discord = TV2 (PENDING) |
| Evidence Store: schema, ingestion, dedup, lifecycle, TTL, seed (public + simulation) | **TV3** | Ghi từ Guardian = PENDING |
| Name Inquiry mock | **TV3** | |
| SafeTransfer engine: claim providers, policy, `POST /v1/transfer-risk` | **TV3** | |
| Trang web tối giản **SafeTransfer Check** (HTML tĩnh do FastAPI phục vụ) | **TV3** | Để demo độc lập. Banking simulator đẹp + Discord Activity = TV2 |
| FastAPI app shell, config, auth API key, logging, health | **TV3** | |
| Golden fixtures, eval set, đo latency | **TV3** | |
| Tích hợp end-to-end, demo script | **TV3 chủ trì** | |
| Guardian engine, `POST /v1/screen`, trích claim từ tin nhắn | TV1 | |
| Discord bot, tier UI, mod review, `/setup`, nút "Báo cáo", nút "Chuyển khoản", banking simulator UI | TV2 | |

Đánh đổi: khối lượng TV3 tăng. Cách giảm: UI chỉ ở mức tối giản, registry 10 tổ chức trước, auth chỉ dùng API key.

## 6. Trạng thái contract

Quy ước nhãn: **FROZEN** (đã chốt cả nhóm) · **DRAFT-TV3** (TV3 tự quyết, có thể đổi, phải thông báo khi đổi) · **PENDING-NEED-CONFIRM-CONTRACT** (cần chốt với TV khác; code dùng stub/adapter).

| ID | Hạng mục | Trạng thái | Cần chốt với |
| --- | --- | --- | --- |
| K01 | Quy tắc normalise STK/bank, công thức `recipient_key` | DRAFT-TV3 | TV1 (xác nhận sẽ import `fw_common`) |
| K02 | VietQR parse/encode/decode + `POST /v1/qr/decode` | DRAFT-TV3 | TV2 (định dạng upload ảnh) |
| K03 | `POST /v1/transfer-risk` request/response | DRAFT-TV3 | TV2 (trường UI cần hiển thị) |
| K04 | Decision table (Đỏ/Vàng/Xám/Xanh) | DRAFT-TV3 | Cả nhóm: `amount_band` có đổi verdict không; group mismatch là hard hay soft |
| K05 | Evidence schema + `POST /v1/evidence` | DRAFT-TV3 | — |
| K06 | Trường evidence gửi từ Guardian / mod confirm / nút Báo cáo | PENDING-NEED-CONFIRM-CONTRACT | TV1, TV2 |
| K07 | Claim context từ Discord (`screening_id`, cache 30 phút, `ClaimContext`) | PENDING-NEED-CONFIRM-CONTRACT | TV1 |
| K08 | `POST /v1/screen` | PENDING-NEED-CONFIRM-CONTRACT | TV1 (owner) |
| K09 | Payload Group Trust từ `/setup` | PENDING-NEED-CONFIRM-CONTRACT | TV2 |
| K10 | Handoff nút "Chuyển khoản" → simulator (truyền `screening_id`) | PENDING-NEED-CONFIRM-CONTRACT | TV2 |
| K11 | Stack chung (Python 3.11 + FastAPI + Pydantic v2, một app hay hai process) | PENDING-NEED-CONFIRM-CONTRACT | TV1, TV2 |
| K12 | Hosting/deploy cho demo | PENDING-NEED-CONFIRM-CONTRACT | Cả nhóm |

Tài liệu đầy đủ: Claude Doc "FraudWeave — Integration Contract v0.1" (export markdown vào `sources/contract-v0.1.md`). §3 của doc đó đã được §5 ở đây thay thế.

## 7. Decision table (bản DRAFT-TV3, thứ tự match đầu tiên thắng)

| # | Điều kiện | Verdict |
| --- | --- | --- |
| 0 | Không normalise được người nhận | HTTP 422 |
| 1 | Khớp official/declared **và** có evidence verified | Vàng (`registry_conflict`, đưa vào review) |
| 2 | Có contradiction cứng C2/C3/C5/C6 | Đỏ |
| 3 | Khớp official/declared | Xanh (không bao giờ dùng chữ "an toàn") |
| 4 | Có evidence chưa xác minh, tín hiệu mềm, hoặc tổ chức không có trong registry | Vàng |
| 5 | Còn lại | Xám ("chưa có dữ liệu, không có nghĩa là an toàn") |

Simulation evidence chỉ được tính khi `mode=demo`. Báo cáo chưa xác minh không bao giờ đưa verdict lên Đỏ.

## 8. Kiến trúc thư mục (3 nơi, tách bạch)

| Thư mục | Vai trò | Quy tắc |
| --- | --- | --- |
| `C:\Users\Dell\Downloads\MomoTechDay\Implements\Agent Workspace` | Harness 3-level: CLAUDE.md, AGENTS.md, MAP.md, SPEC, PLAN, context, rules, skills, agents, nhật ký | Không chứa code sản phẩm. Gọn để agent quét và nắm bối cảnh nhanh |
| `C:\Users\Dell\Downloads\MomoTechDay\Implements\Source code-SafeTransfer\FraudWaveAI-Momo` | Git repo sản phẩm (đã clone, hiện chỉ có README) | Chứa **toàn bộ** pipeline end-to-end: source, contracts, fixtures, migrations, seed, unit/contract tests, scripts, docs. Sạch, rõ ràng, push được |
| `C:\Users\Dell\Downloads\MomoTechDay\Implements\Test-for-agent` | Sandbox của agent: test thử, script khám phá, benchmark nháp, output tạm | **Không** chứa thứ gì pipeline hoặc CI cần để chạy. Có thể xoá bất cứ lúc nào mà repo vẫn chạy |

Quy tắc phân xử: *"Nếu xoá Test-for-agent mà repo vẫn chạy và CI vẫn pass thì đặt đúng chỗ."* Test chính thức (unit, contract, golden) **phải nằm trong repo** (`tests/`), vì đó là một phần của pipeline.

## 9. Stack đề xuất cho repo (K11 còn PENDING; TV3 dùng tạm)

- Python 3.11, `uv` để quản lý môi trường, FastAPI, Pydantic v2, SQLAlchemy 2 + Alembic, Postgres (Supabase; local có thể dùng Docker Postgres).
- QR: `opencv-python-headless` (`QRCodeDetector`) để decode, tránh phụ thuộc zbar trên Windows; `qrcode` + `pillow` để encode. TLV + CRC-16/CCITT-FALSE tự viết, có test.
- Chất lượng: `pytest`, `ruff`, `mypy` (mức vừa phải).
- Secrets: `.env` (không commit), `.env.example` (commit). `FW_HMAC_KEY`, `DATABASE_URL`, `API_KEYS`.

## 10. Kế hoạch (bám gate)

1. 25–26/11: harness 3-level + scaffold repo + CONTRACT_STATUS.
2. 26/11–29/11 (**G1**): `fw_common` (normalise + VietQR parse/encode/decode) + DB schema + seed + `/v1/transfer-risk` chạy được với golden G1, R1, Y1, E1.
3. 29/11–1/12: đủ 11 golden cases, evidence ingestion, name inquiry mock, trang SafeTransfer Check, eval set (freeze 1/12).
4. 1/12–5/12 (**G2**): tích hợp: chốt K06–K10, thay stub bằng adapter thật, TV2 gọi API thật, luồng Discord → "Chuyển khoản" → verdict.
5. 5/12–9/12 (**G3**): latency, báo cáo eval, registry 50 tổ chức, demo script. Hỗ trợ TV1/TV2 nếu cần.

## 11. Lưu ý cần kiểm chứng (chưa xác minh)

- Chi tiết VietQR/EMVCo (tag 38 GUID, service code, tag 62 memo, CRC) cần đối chiếu với spec chính thức của NAPAS trước khi coi là FROZEN.
- Dữ liệu registry trong ví dụ (EVN, hotline...) là placeholder.
