# Handoff cho TV1 — `fw_common` contracts, ClaimContext, decision table v2 (27/11)

## Tóm tắt
- Đã sẵn sàng: Pydantic models dùng chung (`fraudweave.common.contracts`), `contracts/openapi.json`, 12 golden fixture cho SafeTransfer.
- Nguồn: repo `FraudWaveAI-Momo`, nhánh `tv3/contracts-golden-fixtures`, commit `c5e92c4`. 118 test pass.
- **Chưa có:** normaliser STK/bank + `recipient_key` (T-07/T-08, K01), parser VietQR (T-09/T-10), evidence ingestion (T-21). TV1 chưa import được normaliser.

## Models TV1 dùng
File `src/fraudweave/common/contracts.py` (= `fw_common.contracts` trong contract):

| Model | Dùng cho |
| --- | --- |
| `ClaimContext` | claim trích từ tin nhắn (`source = discord_message`), gửi kèm `screening_id` |
| `Contradiction` | C1–C9; validator bắt buộc C1–C6 = `hard`, C7–C9 = `soft` |
| `Indicator`, `EvidenceRecord` | indicator và evidence |
| `EvidenceCreateRequest` / `EvidenceCreateResponse` | `POST /v1/evidence` (loader của TV1) |

- Request models (`ClaimContext`, `EvidenceCreateRequest`, ...) **từ chối field lạ**; response models bỏ qua field lạ.
- `EvidenceRecord.first_seen / last_seen / expires_at` phải là datetime **có timezone** (ví dụ `...Z`); datetime naive bị từ chối.
- `recipient_key` = `"rk_" + 32 hex` theo contract §4.1 (code tính key chưa có, T-08).

## Endpoint
| Method | Path | Trạng thái |
| --- | --- | --- |
| POST | `/v1/screen` | Của TV1 (K08); TV3 chỉ chừa chỗ mount router |
| POST | `/v1/evidence` | Models có, **route chưa có** (T-21) |
| GET | `/v1/registry/orgs?q=` | Chưa có (T-22) |

Header và error envelope chung: `X-API-Key`, `X-Request-Id`; mọi lỗi trả `{ "error": { "code", "message", "request_id" } }`. Lỗi validation: thiếu field/sai kiểu/field lạ → **400 `bad_request`**, chỉ lỗi người dùng nhập sai người nhận → 422 (lệch mặc định FastAPI).

## Ví dụ (chép từ fixture `r4_police_never_asks_transfer.json`)
```json
{
  "recipient": { "bank": "VCB", "account_no": "9990000000009" },
  "amount_band": "1m_10m",
  "claim_context": { "claimed_org_raw": "Cong an-demo", "claimed_role": "police", "source": "user_declared" },
  "mode": "live"
}
```
Expected: `red`, `cancel`, phải chứa C6 + C2, không được chứa C3, C5. Setup: org `cong_an_demo` có `never_asks = ["transfer"]`.

## Thay đổi ảnh hưởng TV1 (proposed, cần TV1 xác nhận)
- **C7–C9 (soft signals) không ảnh hưởng verdict SafeTransfer MVP.** Guardian vẫn có thể gửi `soft_signals` trong `ClaimContext` (enum giữ nguyên, không breaking), nhưng SafeTransfer bỏ qua. Có implement thêm hay không sẽ chốt sau khi TV1/TV2 xong phần của mình.
- **Bảng K04 v2** (first match wins): 0 → 422; 1 hard C2/C3/C6 → red; 2 khớp official/declared + evidence verified → amber `registry_conflict`; 3 C5 → red; 4 khớp official/declared → green; 5 report chưa xác minh hoặc org không có trong registry → amber; 6 → grey.
- **C3**: holder `personal` **và** (org resolve được trong registry **hoặc** role org_staff/bank_staff/police). Tên org lạ một mình không bắn C3; `unknown` không bao giờ bắn C3.
- **C6**: registry chỉ hỗ trợ token `never_asks = "transfer"`; không có `personal_transfer`. Chưa có trường phạm vi áp dụng.
- Resolve `claimed_org_raw` → `org_id`: khớp alias **chính xác** sau chuẩn hoá (chữ thường, bỏ dấu, gộp khoảng trắng), không khớp mờ.

## Fixtures
- `contracts/fixtures/transfer/` — 12 ca synthetic (STK dải `999000…`), cách đọc `*_include` / `*_forbidden` ghi trong `contracts/CONTRACT_STATUS.md`.
- `contracts/fixtures/normalize/` — **chưa có** (T-13): bộ "1 người nhận viết 6 cách → 1 `recipient_key`" để TV1 chạy trong CI của mình.

## Cách chạy local
```
git fetch && git switch tv3/contracts-golden-fixtures
uv sync
uv run pytest
```

## Việc TV1 cần chốt
| Kxx | Câu hỏi | Hạn |
| --- | --- | --- |
| K01 | TV1 import `fraudweave.common` (không tự viết normaliser)? Đóng gói thế nào (cùng repo / package)? | Trước 30/11 (T-06d) |
| K04 | Đồng ý bảng v2 + C7–C9 không vào verdict MVP? | Trước 30/11 |
| K07 | Các trường `ClaimContext`, định dạng `screening_id`, cache 30 phút do ai giữ? | Trước 30/11 |
| K06 | Payload evidence từ Guardian (tạm theo contract §5.3) | Trước G2 5/12 |
| K08 | Engine đọc registry/evidence qua repository của TV3 hay cache nạp lúc khởi động? | Trước 30/11 |
| K11 | Một FastAPI app hay hai process; stack Python 3.11 + FastAPI + Pydantic v2? | Trước 30/11 |

## Đang là stub
- `# PENDING-NEED-CONFIRM-CONTRACT(K07)` — `ClaimContext` và `TransferRiskRequest.screening_id` (`common/contracts.py`).
- `# PENDING-NEED-CONFIRM-CONTRACT(K06)` — `EvidenceCreateRequest`.
- K08: TV3 không implement `/v1/screen`.
