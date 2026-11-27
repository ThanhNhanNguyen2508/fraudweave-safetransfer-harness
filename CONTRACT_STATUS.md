# CONTRACT_STATUS — K01–K12

> **Nơi DUY NHẤT** theo dõi trạng thái contract (D-006). Bản trong REPO chỉ là bản sao gọn, phải đồng bộ theo file này.
> Nhãn: **FROZEN** · **DRAFT-TV3** · **PENDING-NEED-CONFIRM-CONTRACT**. Quy tắc đổi: `.claude/rules/contract-pending.md`.
> Marker trong code: `# PENDING-NEED-CONFIRM-CONTRACT(Kxx): <mô tả>`. Skill `contract-audit` đối chiếu marker với bảng này.

| ID | Hạng mục | Trạng thái | Cần chốt với | Stub/adapter trong code | Ngày cập nhật |
| --- | --- | --- | --- | --- | --- |
| K01 | Quy tắc normalise STK/bank, công thức `recipient_key` | DRAFT-TV3 | TV1 (xác nhận import `fw_common`) | Không cần stub. Có `common/banks.py` + `data/bank_aliases.json` + `text.py` (T-07, chưa commit); `normalize.py`, `hashing.py` (T-08, chưa commit; chi tiết chọn ở D-044 chờ TV3 duyệt) | 2026-11-25 |
| K02 | VietQR parse/encode/decode + `POST /v1/qr/decode` | DRAFT-TV3 | TV2 (định dạng upload ảnh) | Dự kiến `common/vietqr/*`, `api/routers/qr.py`; tạm nhận multipart image (chưa có code). Parser: `common/vietqr/parse.py` (T-10, chưa commit), fixture `contracts/fixtures/qr/`. Nguồn: `REPO/docs/vietqr-spec-notes.md` (T-07a) — quy tắc tag mới ở mức SECONDARY; **chưa được lên FROZEN** cho tới khi có spec NAPAS / TCCS 3:2018 chính thức | 2026-11-27 |
| K03 | `POST /v1/transfer-risk` request/response | DRAFT-TV3 | TV2 (trường UI cần hiển thị) | Models trong `common/contracts.py` (T-05, chưa commit); router `api/routers/transfer.py` chưa có (T-19). `holder_type` ∈ personal/organization/unknown (D-031); `reason_codes: list[str]` + `demo_data_used` trong response (D-032, sẽ thành enum: báo TV2); 422 chỉ cho lỗi nhập sai, còn lại 400 (D-033, lệch mặc định FastAPI: báo TV2); request `forbid`, response `ignore` (D-034) | 2026-11-26 |
| K04 | Decision table Đỏ/Vàng/Xám/Xanh | DRAFT-TV3 | Cả nhóm: `amount_band` có đổi verdict không; group mismatch hard hay soft | **Bảng MVP v2 proposed** (D-035…D-039): thứ tự mới, C3 cần org resolve hoặc role, C6 chỉ token `transfer`, bỏ C7–C9 khỏi verdict (reserved). Chờ TV1/TV2 duyệt qua handoff T-06c; chưa có code (`safetransfer/policy.py`, T-17) | 2026-11-27 |
| K05 | Evidence schema + `POST /v1/evidence` | DRAFT-TV3 | — | Models `EvidenceCreateRequest/Response` trong `common/contracts.py` (T-05); `evidence/ingestion.py`, router chưa có | 2026-11-26 |
| K06 | Trường evidence gửi từ Guardian / mod confirm / nút Báo cáo | PENDING-NEED-CONFIRM-CONTRACT | TV1, TV2 | Dự kiến `integrations/guardian_adapter.py` (interface + stub), dùng tạm payload contract §5.3. Marker K06 ở `EvidenceCreateRequest` trong `common/contracts.py`; adapter chưa có | 2026-11-26 |
| K07 | Claim context từ Discord (`screening_id`, cache 30 phút, `ClaimContext`) | PENDING-NEED-CONFIRM-CONTRACT | TV1 | Dự kiến `safetransfer/claim_providers.py` + screening cache stub in-memory. Marker K07 ở `ClaimContext` và `TransferRiskRequest.screening_id` trong `common/contracts.py`; provider/cache chưa có | 2026-11-26 |
| K08 | `POST /v1/screen` | PENDING-NEED-CONFIRM-CONTRACT | TV1 (owner) | TV3 không implement; chỉ chừa chỗ mount router trong app shell (chưa có code) | 2026-11-25 |
| K09 | Payload Group Trust từ `/setup` | PENDING-NEED-CONFIRM-CONTRACT | TV2 | Dự kiến `api/routers/groups.py` theo contract §5.4 làm bản tạm (chưa có code) | 2026-11-25 |
| K10 | Handoff nút "Chuyển khoản" → simulator (truyền `screening_id`) | PENDING-NEED-CONFIRM-CONTRACT | TV2 | Phía TV3: `screening_id` optional trong request `transfer-risk`; web Check nhận qua query param [GIẢ ĐỊNH] (chưa có code) | 2026-11-25 |
| K11 | Stack chung (Python 3.11 + FastAPI + Pydantic v2; một app hay hai process) | PENDING-NEED-CONFIRM-CONTRACT | TV1, TV2 | TV3 dùng tạm stack brief §9, một FastAPI app (chưa có code) | 2026-11-25 |
| K12 | Hosting/deploy cho demo | PENDING-NEED-CONFIRM-CONTRACT | Cả nhóm | Chạy local; chưa có deploy | 2026-11-25 |

## Câu hỏi mở cần chốt ở buổi freeze (contract §10)

| Câu hỏi | Liên quan |
| --- | --- |
| `amount_band` có bao giờ đổi verdict (ví dụ Xám → Vàng khi > 50M) hay chỉ đổi lời khuyên? | K04 |
| Group `declared_accounts_only` mismatch là hard (Đỏ) hay soft (Vàng) khi role được claim không phải anchor? | K04, K09 |
| ~~Xanh có thắng tín hiệu mềm C7–C9 không?~~ Đã gác: C7–C9 bỏ khỏi verdict MVP (D-039). **Chờ TV1/TV2 xong rồi mới chốt có implement thêm C7–C9 không** | K04, K07 |
| ~~Org không resolve + role org_staff/bank_staff/police + personal~~ TV3 chốt: C3 bắn theo nhánh role (D-041); vẫn thuộc bảng v2 proposed, chờ nhóm | K04 |
| `never_asks` cần trường phạm vi áp dụng (role/kênh/loại giao dịch) không? MVP: không, `transfer` áp cho mọi yêu cầu (D-038) | K04, registry |
| Ai được retract evidence trong demo: chỉ admin TV3 hay cả server mod? | K05, K06 |
| Engine TV1 đọc Postgres qua repository của TV3, hay cache in-process nạp lúc khởi động? | K08, K11 |
| Một FastAPI app cho cả hai tính năng hay hai process? | K11 |
| Ai ký duyệt `source_url` của registry 50 tổ chức? | registry (G3) |

## Lịch sử thay đổi trạng thái

| Ngày | ID | Từ → Sang | Ai | Ghi chú |
| --- | --- | --- | --- | --- |
| 2026-11-25 | K01–K12 | — → trạng thái khởi tạo | TV3 | Lấy từ brief §6 |
| 2026-11-26 | K03, K05, K06, K07 | giữ nguyên trạng thái | agent | T-05: thêm models + marker K06/K07; chưa đổi nhãn |
| 2026-11-27 | K03 | DRAFT-TV3 (giữ) | TV3 | Duyệt D-031…D-034 cho T-05; cần báo TV2: reason_codes sẽ thành enum, mapping 422/400 |
| 2026-11-27 | K01–K12 | giữ nguyên | agent | T-06: tạo bản sao gọn `REPO/contracts/CONTRACT_STATUS.md`; test `test_fixtures_schema.py` bắt mọi marker PENDING trong `src/` phải là PENDING trong bản sao |
| 2026-11-27 | K04 | DRAFT-TV3 (giữ), bảng v2 **proposed** | TV3 | D-035…D-040; đưa vào handoff T-06c cho TV1/TV2; chưa freeze |
| 2026-11-27 | K04 | DRAFT-TV3 (giữ), v2 proposed | TV3 | D-041: C3 nhánh role, A3 không cấm `matches_official_account`, `st-policy-0.2` |
| 2026-11-27 | K02 | DRAFT-TV3 (giữ) | agent | T-07a: không tìm được spec tag-level chính thức công khai; chặn FROZEN cho tới khi TV3 lấy được bản NAPAS/TCCS 3:2018 |
| 2026-11-27 | K02 | DRAFT-TV3 (giữ) | TV3 | D-042: phạm vi MVP `QRIBFTTA` + 10 BIN; FROZEN vẫn chặn (EMVCo v1.1 cần login; bản NAPAS trên kiemtrabank.com không phân giải DNS) |
| 2026-11-27 | K02 | DRAFT-TV3 (giữ) | TV3 | D-043: chốt VQ-Q2/Q3/Q5/Q7; T-07a done; vẫn chưa FROZEN (không có spec NAPAS/EMVCo bản chính thức đọc được) |
| 2026-11-27 | K01 | DRAFT-TV3 (giữ) | agent | T-08: hiện thực normalise + `recipient_key`; D-044 đề xuất 4 chi tiết chưa có trong contract (encoding khoá, độ dài khoá tối thiểu, khoảng trắng Unicode, đầu vào đã chuẩn hoá); cần báo TV1 hàm dùng chung |
| 2026-11-27 | K01 | DRAFT-TV3 (giữ) | agent | Review T-08 PASS. Cần báo TV1: dùng `hmac_key_from_env()` + `normalize_account_number()` trước `recipient_key()`; `.env.example` cần sửa câu chữ "hex" (ngoài phạm vi T-08) |
| 2026-11-27 | K01 | DRAFT-TV3 (giữ) | TV3 | D-045 duyệt D-044; `.env.example` đã sửa mô tả `FW_HMAC_KEY`; handoff TV1 viết xong, chờ TV3 gửi |
| 2026-11-27 | K02 | DRAFT-TV3 (giữ) | agent | T-10: parser + 17 fixture QR; D-046 đề xuất các kiểm tra lõi; vẫn chưa FROZEN (nguồn tag SECONDARY). `POST /v1/qr/decode` (T-12) vẫn chờ TV2 xác nhận định dạng upload |
| 2026-11-27 | K02 | DRAFT-TV3 (giữ) | TV3 | D-047 duyệt D-046 (kiểm tra lõi, tag 01 bắt buộc) và chấp nhận amount có số 0 đầu; vẫn chưa FROZEN |
