# Handoff cho TV2: `POST /v1/transfer-risk` chạy thật + OpenAPI cập nhật (27/11)

> Từ TV3. Bổ sung cho handoff `2026-11-27-tv2-transfer-risk-contract.md` (models và 12 fixture). Contract K03 vẫn **DRAFT-TV3**. TV3 gửi; agent không gửi thay.

## Tóm tắt
- Endpoint đã chạy thật trên Postgres local: **12/12 golden pass qua HTTP** (G1, G2, R1–R4, A1–A4, Y1, E1). Có thể chuyển simulator từ mock sang API thật khi TV3 báo commit/push.
- `contracts/openapi.json` đã sinh lại: có `POST /v1/transfer-risk` kèm security scheme `X-API-Key` và **đủ các response lỗi** (xem dưới). Hãy sinh lại client/mock từ file này, đừng dùng bản cũ (chỉ có 200 và `HTTPValidationError` mặc định).
- Chưa có: rate limit 60 req/phút/key và `policy_version` trong `/v1/health` (T-25), `POST /v1/qr/decode` (T-12), cache `screening_id` thật (T-26, đang là stub).

## Gọi API
- Header bắt buộc: `X-API-Key` (mỗi client một key do TV3 cấp, ví dụ `simulator`). Tuỳ chọn: `X-Request-Id` (chỉ chấp nhận `[A-Za-z0-9_-]{1,64}`, sai định dạng thì server tự sinh). **Mọi response (cả thành công) trả lại `X-Request-Id`.**
- Body: `TransferRiskRequest` như trong contract §5.1. Nếu gửi cả `vietqr_payload` lẫn `bank` + `account_no` thì **payload thắng**.
- `screening_id` thắng `claim_context`. `screening_id` lạ hoặc hết hạn: bỏ qua, thêm reason code `screening_expired`, dùng `claim_context` nếu có (D-050). Hiện chưa có gì ghi vào cache nên mọi `screening_id` đều là "lạ".

## Mã lỗi (envelope `{"error": {"code", "message", "request_id"}}`, schema `ErrorResponse`)
| HTTP | `code` | Khi nào |
| --- | --- | --- |
| 400 | `bad_request` | Thiếu trường, sai kiểu, **trường lạ** (request `forbid`), body không phải JSON hợp lệ |
| 401 | `invalid_api_key` | Thiếu hoặc sai `X-API-Key`. Đến **trước** validate body JSON hợp lệ |
| 422 | `invalid_recipient` / `unknown_bank` / `invalid_qr` | Người dùng nhập STK/ngân hàng/QR không dùng được |
| 503 | `internal_error` | Không truy cập được dữ liệu (DB treo/không kết nối), thông báo "tạm thời không kiểm tra được" |
| 500 | `internal_error` | Lỗi không lường trước hoặc thiếu cấu hình |

**Mọi lỗi 5xx: client hiển thị "Không kiểm tra được", không bao giờ hiển thị Xanh.** Thông báo lỗi không chứa lại STK, payload hay memo.

## Hành vi đáng chú ý của response
- `claim_used`: có khi request có claim hiệu lực (org, role hoặc group); `null` nếu không claim gì. `claimed_org_id` chỉ có khi org tra được trong registry.
- `official_alternative`: chỉ khi org được claim có trong registry.
- `contradictions[]`: chỉ mâu thuẫn **hiệu lực**; verdict khác đỏ thì rỗng (D-037). Ca A3 (tài khoản official có evidence verified): Vàng + `registry_conflict`, `evidence_summary.verified ≥ 1`, không có C5.
- `name_inquiry`: luôn có, `source = "mock"`. **Đây là dữ liệu demo, không phải xác minh ngân hàng thật.** Tài khoản không có trong mock: `holder_type = "unknown"`, `holder_name_masked = "***"`.
- `reason_codes` là danh sách mã (chưa là enum, D-032). Câu tiếng Việt cho `contradictions` nằm sẵn trong `message_vi`; với reason code không phải contradiction, UI tự ánh xạ mã sang câu chữ (bảng copy ở `common/reasons.py`, đừng tự viết lại ý nghĩa; Xanh không dùng chữ "an toàn").
- `evidence`: tối đa 3 bản ghi, chỉ trường hiển thị. `demo_data_used = true` khi evidence mô phỏng được tính (chỉ ở `mode=demo`).
- Bảng quyết định là v2 + D-048 (ghép cặp claim–tài khoản, miễn C3 khi khớp official). Xem `deliverables/handoff/2026-11-27-k04-d048-claim-pairing.md` nếu chưa đọc.

## Dữ liệu demo cần biết
- Seed synthetic (STK dải `999000…`) phủ 12 golden: org `EVN-demo`, `VCB-demo`, `Cong an-demo`; group `g_demo_lop5a` (G2, R3). Chỉ dùng cho dev/demo.
- Cách gọi các ca: dùng thẳng `request` trong `contracts/fixtures/transfer/*.json`.

## TV2 cần phản hồi
1. Xác nhận envelope lỗi và OpenAPI mới dùng được để sinh client (K03).
2. Chuẩn bị chuyển từ mock sang API thật khi TV3 báo: base URL, `X-API-Key` cấp sau (K11/K12 còn chưa chốt hosting).
3. Cách hiển thị reason code không phải contradiction (K03).
