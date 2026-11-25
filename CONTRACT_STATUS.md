# CONTRACT_STATUS — K01–K12

> **Nơi DUY NHẤT** theo dõi trạng thái contract (D-006). Bản trong REPO chỉ là bản sao gọn, phải đồng bộ theo file này.
> Nhãn: **FROZEN** · **DRAFT-TV3** · **PENDING-NEED-CONFIRM-CONTRACT**. Quy tắc đổi: `.claude/rules/contract-pending.md`.
> Marker trong code: `# PENDING-NEED-CONFIRM-CONTRACT(Kxx): <mô tả>`. Skill `contract-audit` đối chiếu marker với bảng này.

| ID | Hạng mục | Trạng thái | Cần chốt với | Stub/adapter trong code | Ngày cập nhật |
| --- | --- | --- | --- | --- | --- |
| K01 | Quy tắc normalise STK/bank, công thức `recipient_key` | DRAFT-TV3 | TV1 (xác nhận import `fw_common`) | Không cần stub. Dự kiến `src/fraudweave/common/normalize.py`, `hashing.py` (chưa có code) | 2026-11-25 |
| K02 | VietQR parse/encode/decode + `POST /v1/qr/decode` | DRAFT-TV3 | TV2 (định dạng upload ảnh) | Dự kiến `common/vietqr/*`, `api/routers/qr.py`; tạm nhận multipart image (chưa có code) | 2026-11-25 |
| K03 | `POST /v1/transfer-risk` request/response | DRAFT-TV3 | TV2 (trường UI cần hiển thị) | Dự kiến `common/contracts.py`, `api/routers/transfer.py` (chưa có code) | 2026-11-25 |
| K04 | Decision table Đỏ/Vàng/Xám/Xanh | DRAFT-TV3 | Cả nhóm: `amount_band` có đổi verdict không; group mismatch hard hay soft | Dự kiến `safetransfer/policy.py`; mặc định theo SPEC §4 (amount không đổi verdict, group mismatch = hard C2) (chưa có code) | 2026-11-25 |
| K05 | Evidence schema + `POST /v1/evidence` | DRAFT-TV3 | — | Dự kiến `evidence/ingestion.py`, `api/routers/evidence.py` (chưa có code) | 2026-11-25 |
| K06 | Trường evidence gửi từ Guardian / mod confirm / nút Báo cáo | PENDING-NEED-CONFIRM-CONTRACT | TV1, TV2 | Dự kiến `integrations/guardian_adapter.py` (interface + stub), dùng tạm payload contract §5.3 (chưa có code) | 2026-11-25 |
| K07 | Claim context từ Discord (`screening_id`, cache 30 phút, `ClaimContext`) | PENDING-NEED-CONFIRM-CONTRACT | TV1 | Dự kiến `safetransfer/claim_providers.py` + screening cache stub in-memory (chưa có code) | 2026-11-25 |
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
| Xanh có thắng tín hiệu mềm C7–C9 không? (hiện tại: có, vì luật 3 đứng trước luật 4) | K04 |
| Ai được retract evidence trong demo: chỉ admin TV3 hay cả server mod? | K05, K06 |
| Engine TV1 đọc Postgres qua repository của TV3, hay cache in-process nạp lúc khởi động? | K08, K11 |
| Một FastAPI app cho cả hai tính năng hay hai process? | K11 |
| Ai ký duyệt `source_url` của registry 50 tổ chức? | registry (G3) |

## Lịch sử thay đổi trạng thái

| Ngày | ID | Từ → Sang | Ai | Ghi chú |
| --- | --- | --- | --- | --- |
| 2026-11-25 | K01–K12 | — → trạng thái khởi tạo | TV3 | Lấy từ brief §6 |
