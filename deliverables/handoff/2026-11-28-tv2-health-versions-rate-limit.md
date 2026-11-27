# Handoff TV2 — health có version, rate limit 429 (T-25) · 2026-11-28

## Tóm tắt
- Gate G1 của SafeTransfer **đạt** (28/11). `POST /v1/transfer-risk` pass đủ 12 golden trên Postgres local.
- Mới ở T-25: `GET /v1/health` trả thêm `policy_version`, `contract_version`. `transfer-risk` có giới hạn **60 request/phút cho mỗi API key**, vượt thì trả **429**.
- Nhánh `tv3/api-health-ratelimit` (TV3 commit). OpenAPI mới ở `contracts/openapi.json`.

## Endpoint / API

| Method | Path | Header | Mã lỗi |
| --- | --- | --- | --- |
| GET | `/v1/health` | không cần key, không bị rate limit | — |
| POST | `/v1/transfer-risk` | `X-API-Key` (bắt buộc), `X-Request-Id` (tuỳ chọn, server trả lại) | 400, 401, 422, **429 (mới)**, 500, 503 |

Thứ tự xử lý: kiểm key (401) → rate limit (429) → validate body (400/422) → verdict.
- Request sai key **không** bị đếm.
- Body JSON hợp lệ nhưng sai schema **có** bị đếm.
- Body không phải JSON vẫn trả 400 trước bước kiểm key, như cũ.

## Ví dụ (chạy thật trên code hiện tại)

`GET /v1/health` → 200
```json
{"status": "ok", "app_version": "0.1.0", "policy_version": "st-policy-0.2", "contract_version": "0.1"}
```

`POST /v1/transfer-risk` khi đã vượt 60 request/phút → 429, header `Retry-After: 60`
```json
{"error": {"code": "rate_limited", "message": "Rate limit exceeded", "request_id": "req_example429"}}
```
- `Retry-After` là số giây nguyên ≥ 1 cần chờ, được khai trong OpenAPI.
- 429 **không phải** verdict. Simulator nên hiện "Thử lại sau N giây", không hiện Xanh và cũng không hiện "Không kiểm tra được" như khi gặp 5xx.

## Fixtures dùng làm mock
Không đổi: `contracts/fixtures/transfer/*.json` (12 golden) vẫn đúng. Không có fixture mới cho 429; mock phía TV2 có thể tự trả envelope ở trên.

## Cách chạy local
```
uv sync
uv run uvicorn fraudweave.api.main:app --reload
```
Cần `.env` có `DATABASE_URL`, `FW_HMAC_KEY`, `API_KEYS=<client>:<key>,...` (xem `.env.example`). Riêng `/v1/health` chạy được khi chưa có cấu hình.

## Việc bạn cần chốt

| Kxx | Câu hỏi | Hạn |
| --- | --- | --- |
| K03 | 60 request/phút cho mỗi key có đủ cho simulator và web demo không? Nếu nhiều màn hình dùng chung một key thì cần thêm key riêng | họp T-06d, trước 30/11 |
| K03 | UI hiển thị 429 thế nào (đếm ngược theo `Retry-After`?) | trước G2 (5/12) |
| K12 | Deploy demo chạy **1 worker** (rate limit đếm trong bộ nhớ một process, D-051) | khi chốt K12 |

Các câu hỏi K02, K06, K09, K10, K11 từ handoff ngày 27/11 vẫn còn nguyên.

## Đang là stub (liên quan TV2)
- `POST /v1/evidence` **chưa có** (T-21). Khi có sẽ dùng cùng rate limit (contract §7).
- K07 (`screening_id`): `InMemoryScreeningCache` là stub (`safetransfer/claim_providers.py`), marker `PENDING-NEED-CONFIRM-CONTRACT(K07)`.
