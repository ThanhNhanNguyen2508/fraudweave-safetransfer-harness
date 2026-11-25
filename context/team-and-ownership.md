# Team và ownership

> Nguồn: brief §5 (thay thế contract §3). Cập nhật 2026-11-25.
> "Owner" = viết và chịu trách nhiệm. "Consumer" = phải gọi đúng như contract.

## Thành viên

| Vai trò | Người | Phụ trách chính |
| --- | --- | --- |
| **TV1** | [CHƯA KIỂM CHỨNG: tên] | Guardian detection engine: `engine.screen()` (Claim-Check B1–B5 + risk scorer), PhoBERT/ONNX, `POST /v1/screen`, trích claim từ tin nhắn, Guardian eval |
| **TV2** | [CHƯA KIỂM CHỨNG: tên] | Discord bot, tier UI, mod review, `/setup`, nút "Báo cáo", nút "Chuyển khoản", banking simulator UI, Discord Activity |
| **TV3** (người dùng harness) | Rykanny | Backend & Fraud Intelligence: toàn bộ SafeTransfer end-to-end + app shell + dữ liệu |

Ghi chú [ĐÃ XÁC MINH 25/11, TV3 xác nhận]: remote của REPO `github.com/ThanhNhanNguyen2508/FraudWaveAI-Momo` là tài khoản GitHub của **chính TV3**, có toàn quyền (bật được branch protection). HARNESS sẽ nằm ở private repo `fraudweave-safetransfer-harness` của cùng tài khoản.

## Module TV3 sở hữu

| Module | Ghi chú |
| --- | --- |
| VietQR: decode ảnh → payload, parse TLV + CRC, encode | Có `POST /v1/qr/decode` để TV2 dùng lại (K02) |
| `fw_common`: normaliser, bank aliases/BIN, `recipient_key`, reason codes, Pydantic contracts | TV1 **import**, không tự viết lại (K01) |
| Org Registry | Seed 10 tổ chức demo do TV3 curate, mở rộng lên 50 ở G3 |
| Group Trust store + API | UI `/setup` thuộc TV2 (K09 PENDING) |
| Evidence Store: schema, ingestion, dedup, lifecycle, TTL, seed (public + simulation) | Ghi từ Guardian là PENDING (K06) |
| Name Inquiry mock | |
| SafeTransfer engine: claim providers, policy, `POST /v1/transfer-risk` | K03, K04 |
| Trang web tối giản **SafeTransfer Check** (HTML tĩnh do FastAPI phục vụ) | Để demo độc lập; simulator đẹp là của TV2 |
| FastAPI app shell, config, auth API key, logging, health | |
| Golden fixtures, eval set, đo latency | |
| Tích hợp end-to-end, demo script | TV3 chủ trì |

## Điểm giao với TV khác (ai cần chốt gì)

| Với | Hạng mục contract | Hướng phụ thuộc |
| --- | --- | --- |
| TV1 | K01 (import `fw_common`), K06 (evidence từ Guardian), K07 (`ClaimContext` + `screening_id`), K08 (`/v1/screen`), K11 (stack) | TV1 dùng thư viện của TV3; TV3 đọc claim context từ TV1 |
| TV2 | K02 (định dạng upload ảnh QR), K03 (trường UI cần hiển thị), K06 (mod confirm / nút Báo cáo), K09 (payload `/setup`), K10 (nút "Chuyển khoản" → simulator), K11 | TV2 gọi API của TV3 và dùng fixtures của TV3 làm mock |
| Cả nhóm | K04 (decision table: `amount_band`, group mismatch hard/soft), K12 (hosting) | |

Đánh đổi đã chấp nhận: khối lượng TV3 tăng. Cách giảm: UI tối giản, registry 10 tổ chức trước, auth chỉ dùng API key.
