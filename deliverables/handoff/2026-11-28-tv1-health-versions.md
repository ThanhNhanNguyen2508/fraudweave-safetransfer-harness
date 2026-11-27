# Handoff TV1 — health có version, Gate G1 đạt (T-25) · 2026-11-28

## Tóm tắt
- Gate G1 của SafeTransfer **đạt** (28/11): `fw_common` (normalise, VietQR parse + CRC), DB, `transfer-risk` pass 12 golden trên Postgres local.
- `GET /v1/health` trả `app_version`, `policy_version`, `contract_version`. **Chưa có `model_version`**: field này là của engine TV1 (K08) nên tôi chưa tự thêm.
- Nhánh `tv3/api-health-ratelimit` (TV3 commit). Hàm `fw_common` không đổi so với handoff 27/11.

## Endpoint / API

| Method | Path | Header | Ghi chú |
| --- | --- | --- | --- |
| GET | `/v1/health` | không cần key | Không bị rate limit |

`GET /v1/health` → 200 (chạy thật trên code hiện tại):
```json
{"status": "ok", "app_version": "0.1.0", "policy_version": "st-policy-0.2", "contract_version": "0.1"}
```
- `contract_version` lấy từ `CONTRACT_VERSION` trong `fraudweave.common.contracts`. Giá trị này chỉ bump khi có breaking change đã họp chốt (contract §10, D-051).
- `policy_version` lấy từ `POLICY_VERSION` trong `fraudweave.safetransfer.policy`.

Có thêm rate limit 60 request/phút cho mỗi API key trên `transfer-risk` (429 `rate_limited` + `Retry-After`). Nếu `/v1/screen` mount chung app (K11), có áp cùng giới hạn hay không là câu hỏi mở bên dưới.

## Fixtures
Không đổi: `contracts/fixtures/transfer/`, `contracts/fixtures/qr/`. `contracts/fixtures/normalize/` **chưa có** (T-13, làm tiếp theo).

## Cách chạy local
```
uv sync
uv run pytest -q
```
Test cần DB thì đặt `FW_TEST_DATABASE_URL` trỏ tới database `*_test`, không đặt thì tự skip.

## Việc bạn cần chốt

| Kxx | Câu hỏi | Hạn |
| --- | --- | --- |
| K08 | Health có trả `model_version` của engine không? Nếu có thì lấy từ hằng nào trong package của bạn? | họp T-06d, trước 30/11 |
| K11 | Nếu mount `/v1/screen` vào cùng app: dùng chung `enforce_rate_limit` (60 request/phút/key) hay đặt giới hạn riêng? | khi chốt K11 |

Các câu hỏi K01, K04, K06, K07 từ handoff ngày 27/11 vẫn còn nguyên.

## Đang là stub (liên quan TV1)
- K07: `ClaimContext`, `TransferRiskRequest.screening_id` (`common/contracts.py`), `InMemoryScreeningCache` (`safetransfer/claim_providers.py`), có marker `PENDING-NEED-CONFIRM-CONTRACT(K07)`.
- K06: `EvidenceCreateRequest` (`common/contracts.py`), có marker `PENDING-NEED-CONFIRM-CONTRACT(K06)`.
