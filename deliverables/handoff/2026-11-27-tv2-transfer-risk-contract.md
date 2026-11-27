# Handoff cho TV2 — contract `POST /v1/transfer-risk` + golden fixtures (27/11)

## Tóm tắt
- Đã sẵn sàng: Pydantic models, `contracts/openapi.json`, error envelope và 12 golden fixture để TV2 dựng mock UI simulator / Discord.
- Nguồn: repo `FraudWaveAI-Momo`, nhánh `tv3/contracts-golden-fixtures`, commit `c5e92c4` (T-05 + T-06). 118 test pass.
- **Chưa có:** endpoint chạy thật (T-19, dự kiến Gate G1 29/11), mock server (T-06b), `POST /v1/qr/decode` (T-12, trước 1/12). Hiện app chỉ có `GET /v1/health`.

## Endpoint / API
| Method | Path | Trạng thái |
| --- | --- | --- |
| GET | `/v1/health` | Chạy được |
| POST | `/v1/transfer-risk` | Có models + OpenAPI schema, **chưa có route** (T-19) |
| POST | `/v1/qr/decode` | Chưa có (T-12) |

- Header (contract §5): `X-API-Key` (mỗi client một key; auth làm ở T-19), `X-Request-Id` (tuỳ chọn; server trả lại đúng giá trị nếu hợp lệ, `[A-Za-z0-9_-]{1,64}`, còn lại tự sinh `req_...`).
- Models: `TransferRiskRequest`, `TransferRiskResponse` trong `src/fraudweave/common/contracts.py`; schema đầy đủ trong `contracts/openapi.json` (`components.schemas`).
- **Error envelope** cho mọi response khác 2xx:
  ```json
  { "error": { "code": "invalid_recipient", "message": "...", "request_id": "req_..." } }
  ```

| HTTP | `code` | UI nên làm |
| --- | --- | --- |
| 400 | `bad_request` | Bug phía caller: sửa code gọi |
| 401 | `invalid_api_key` | Lỗi cấu hình |
| 422 | `invalid_recipient`, `unknown_bank`, `invalid_qr` | Hỏi người dùng nhập lại |
| 429 | `rate_limited` | Thử lại sau `Retry-After` |
| 5xx | `internal_error`, `engine_unavailable` | Trạng thái riêng **"Không kiểm tra được"**, **không bao giờ hiện Xanh** |

**Lệch mặc định FastAPI, cần lưu ý (D-033):** 422 chỉ dành cho lỗi người dùng nhập sai. Thiếu field, sai kiểu JSON, field lạ trả **400 `bad_request`** (FastAPI mặc định trả 422 cho mọi lỗi validation).

## Ví dụ request / expected (chép từ fixture `r2_bank_staff_claim_personal_account.json`)
```json
{
  "recipient": { "bank": "VCB", "account_no": "9990000000004" },
  "amount_band": "1m_10m",
  "claim_context": { "claimed_org_raw": "VCB-demo", "claimed_role": "bank_staff", "source": "user_declared" },
  "mode": "live"
}
```
Expected: `verdict = red`, `recommended_action = cancel`, `contradictions` **phải chứa** C2 + C3, **không được chứa** C5, C6.

Lưu ý: fixture chỉ có phần `expected` rút gọn, **chưa có response đầy đủ** (`recipient.recipient_key`, `name_inquiry`, `message_vi`, `official_alternative`...). Response mẫu đầy đủ lấy từ ví dụ contract §5.1 hoặc `openapi.json`; mock trả response đầy đủ sẽ có ở T-06b.

## Fixtures dùng làm mock
`contracts/fixtures/transfer/` — 12 ca, tất cả synthetic (STK dải `999000…`):

| ID | Verdict | Ghi chú |
| --- | --- | --- |
| G1, G2 | green | khớp tài khoản official / tài khoản quỹ đã khai |
| R1–R4 | red | C5 / C2+C3 / C2 (group) / C6+C2 |
| A1–A4 | amber | unverified report / org không có trong registry / `registry_conflict` / **A4 mới**: org lạ + holder personal |
| Y1 | grey | `no_data` |
| E1 | 422 | `invalid_recipient` |

Cách đọc `expected`: `http_status`, `verdict`, `recommended_action` khớp **chính xác**; `*_include` là mã **phải có**; `*_forbidden` là mã **không được có**; response thật có thể có thêm mã khác. Bản tóm tắt trạng thái contract: `contracts/CONTRACT_STATUS.md`.

## Thay đổi ngữ nghĩa so với contract v0.1 (proposed, cần TV2 xác nhận)
- **Bảng quyết định K04 v2** (first match wins): 0 → 422; 1 hard C2/C3/C6 → red; 2 khớp official/declared + evidence verified → amber `registry_conflict`; 3 evidence verified (C5) → red; 4 khớp official/declared → green; 5 report chưa xác minh hoặc org không có trong registry → amber; 6 còn lại → grey.
- **`contradictions[]` chỉ chứa mâu thuẫn hiệu lực:** thẻ Đỏ liệt kê mọi hard code đúng; thẻ Vàng/Xám/Xanh **không** có hard code. Evidence tìm thấy vẫn nằm trong `evidence_summary` (A3: có evidence verified nhưng không có C5).
- **C7–C9 (soft signals) không ảnh hưởng verdict MVP**; enum vẫn giữ, không breaking.
- `TransferRiskResponse` có thêm `reason_codes: list[str]` (sẽ thành enum trong `fw_common`, sẽ báo trước khi đổi) và `demo_data_used: bool`.
- `NameInquiry.holder_type` ∈ `personal` / `organization` / `unknown`.
- Request model **từ chối field lạ**; response model **bỏ qua field lạ** (server thêm field không làm vỡ client).
- Xanh không bao giờ dùng chữ "an toàn"; Xám: "Chưa có dữ liệu về tài khoản này — điều này không có nghĩa là an toàn."

## Cách chạy local
```
git fetch && git switch tv3/contracts-golden-fixtures
uv sync
uv run pytest
uv run python scripts/export_openapi.py --check
uv run uvicorn fraudweave.api.main:app --reload    # chỉ có /v1/health
```
Không cần `.env` để chạy test.

## Việc TV2 cần chốt
| Kxx | Câu hỏi | Hạn |
| --- | --- | --- |
| K03 | Response hiện tại đủ trường UI cần hiển thị chưa? Có chấp nhận 422/400 như trên? | Họp chốt trước 30/11 (T-06d) |
| K04 | Đồng ý bảng v2, `contradictions[]` chỉ là mâu thuẫn hiệu lực, golden A4? | Trước 30/11 |
| K02 | Định dạng upload ảnh QR cho `/v1/qr/decode` (tạm: multipart image) | Trước 1/12 |
| K06 | Payload từ mod confirm / nút "Báo cáo" (tạm theo contract §5.3) | Trước G2 5/12 |
| K09 | Payload `/setup` cho Group Trust (tạm theo contract §5.4) | Trước G2 5/12 |
| K10 | Nút "Chuyển khoản" → simulator truyền `screening_id` thế nào (tạm: field optional trong request) | Trước G2 5/12 |
| K11 | Một FastAPI app hay hai process? | Trước 30/11 |

## Đang là stub
- `# PENDING-NEED-CONFIRM-CONTRACT(K06)` — `EvidenceCreateRequest` (`common/contracts.py`).
- `# PENDING-NEED-CONFIRM-CONTRACT(K07)` — `ClaimContext`, `TransferRiskRequest.screening_id`.
- K09, K10: chưa có code, sẽ dùng bản nháp contract khi làm T-22 / T-19.
