# Glossary

> Định nghĩa lấy từ brief và contract v0.1. Mục nào nguồn chưa định nghĩa thì ghi rõ [CHƯA CÓ TRONG SOURCES], không tự bịa.

## Sản phẩm và tổ chức dự án

| Thuật ngữ | Nghĩa |
| --- | --- |
| **FraudWeave** | Sản phẩm dự thi MoMo Tech Contest 2026 (Anti-Fraud). MVP gồm Guardian và SafeTransfer |
| **Guardian** | Bot Discord của TV1 + TV2, phát hiện tin nhắn lừa đảo mạo danh theo thời gian thực |
| **SafeTransfer** | Tính năng của TV3: cảnh báo trước khi chuyển khoản, trả verdict kèm bằng chứng và lý do |
| **SafeTransfer Check** | Trang web HTML tĩnh tối giản do FastAPI phục vụ, để demo SafeTransfer độc lập |
| **Shared Fraud Evidence Store** | Supabase Postgres, nơi Guardian và SafeTransfer dùng chung evidence |
| **Gate G1 / G2 / G3** | Mốc chấm: G1 29/11, G2 5/12, G3 9/12/2026. TechDay 11/12/2026. **Đừng nhầm với golden case G1, G2** |
| **K01–K12** | ID các hạng mục contract (xem `CONTRACT_STATUS.md`) |
| **T-xx** | ID task trong `PLAN.md` |
| **FROZEN / DRAFT-TV3 / PENDING-NEED-CONFIRM-CONTRACT** | Nhãn trạng thái contract (xem AGENTS.md §4) |

## Claim và contradiction

| Thuật ngữ | Nghĩa |
| --- | --- |
| **Claim** | Điều người nhận/người gửi *tự xưng*: là tổ chức X hoặc vai trò Y (giáo viên, nhân viên ngân hàng, công an...) |
| **Claim–payee mismatch** | Giá trị cốt lõi của SafeTransfer: claim là X nhưng tài khoản không khớp tài khoản chính thức/đã khai báo của X, hoặc là tài khoản cá nhân |
| **`ClaimContext`** | Model chứa `claimed_org_id`, `claimed_org_raw`, `claimed_role`, `guild_hash`, `source` (`discord_message` / `user_declared` / `transfer_memo`), `soft_signals` (C7–C9) |
| **`claimed_role`** | `org_staff`, `bank_staff`, `police`, `teacher`, `admin`, `treasurer`, `org_support`, `other` |
| **Claim-Check B1–B5** | Các bước trong engine Guardian của TV1. Chi tiết từng bước [CHƯA CÓ TRONG SOURCES] |
| **Contradiction** | Mâu thuẫn tìm thấy, có `code` C1–C9, `severity` (C1–C6 = hard, C7–C9 = soft), `reason_code`, `message_vi`, `evidence_ids` |
| **C1, C4** | Hard contradiction thuộc phía Guardian. Định nghĩa [CHƯA CÓ TRONG SOURCES] |
| **C2** `payee_not_official_account` | Claim trỏ tới org trong registry (hoặc group có `payment_policy = declared_accounts_only` và role teacher/treasurer/admin), nhưng tài khoản không khớp official/declared |
| **C3** `personal_holder_for_org_claim` | `holder_type = personal` **và** (claim resolve được tới org trong registry **hoặc** `claimed_role` ∈ org_staff / bank_staff / police). Org không resolve được tự nó không bắn C3; `unknown` không bao giờ bắn C3 (D-036) |
| **C5** `confirmed_indicator` | Có ≥ 1 evidence **verified** còn hạn trên tài khoản |
| **C6** `org_never_asks_transfer` | `never_asks` của tổ chức được claim có chứa token `transfer` (MVP chỉ hỗ trợ token này, D-038), ví dụ công an yêu cầu chuyển tiền vào "tài khoản an toàn" |
| **C7–C9** | Tín hiệu mềm (soft) đến từ Guardian qua `screening_id`; ý nghĩa từng mã [CHƯA CÓ TRONG SOURCES]. **Reserved, không ảnh hưởng verdict SafeTransfer MVP** (D-039) |

## Verdict và policy

| Thuật ngữ | Nghĩa |
| --- | --- |
| **Verdict** | `red` (Đỏ), `amber` (Vàng), `grey` (Xám), `green` (Xanh). Enum tiếng Anh, TV2 map sang nhãn tiếng Việt |
| **`recommended_action`** | `cancel`, `verify_by_call`, `proceed` |
| **Xanh** | Chỉ khi khớp official/declared account. **Không bao giờ dùng chữ "an toàn"**. Không có claim thì không thể Xanh |
| **Xám** | Không có dữ liệu. Copy: "Chưa có dữ liệu về tài khoản này — điều này không có nghĩa là an toàn." |
| **Decision table** | Bảng luật có thứ tự, first match wins (SPEC.md §4). Nằm trong `safetransfer/policy.py` |
| **`policy_version`** | Phiên bản policy trả trong mọi response, hiện là `st-policy-0.1` (bảng v2 từ T-17: `st-policy-0.2`, D-041) |
| **`registry_conflict`** | Tài khoản khớp official/declared **nhưng** có evidence verified → Vàng và đưa vào review |
| **Report poisoning** | Báo cáo giả để bôi nhọ tài khoản hợp lệ. Chặn bằng quy tắc: báo cáo chưa xác minh tối đa chỉ lên Vàng |
| **`match_official` / `match_declared`** | `recipient_key` nằm trong `official_account_keys` của org / trong declared fund accounts của group |
| **`E_verified` / `E_unverified`** | Số evidence verified / unverified còn hạn trên key. Simulation chỉ tính vào `E_verified` khi `mode=demo` (response có `demo_data_used: true`) |
| **`mode`** | `live` hoặc `demo` |
| **`amount_band`** | `lt_1m`, `1m_10m`, `10m_50m`, `gt_50m`. Trong MVP chỉ đổi lời khuyên, không đổi verdict (câu hỏi mở K04) |
| **Tier T0–T3** | Mức phản ứng của Guardian: 0 im lặng, 1 báo mod, 2 trả lời cảnh báo, 3 xoá + trả lời |

## Recipient và dữ liệu

| Thuật ngữ | Nghĩa |
| --- | --- |
| **STK** | Số tài khoản ngân hàng. Sau khi làm sạch phải là 6–19 chữ số, giữ số 0 đầu |
| **BIN** | Mã ngân hàng 6 chữ số của NAPAS (ví dụ `970436` = VCB) |
| **`recipient_key`** | `"rk_" + hex(HMAC_SHA256(FW_HMAC_KEY, f"acct:{bin}:{account_no}"))[:32]`. Là thứ duy nhất được lưu thay cho STK |
| **`account_masked`** | STK đã che, chỉ hiện 4 số cuối (ví dụ `*********4567`) |
| **`NormalizedRecipient`** | `bank_bin`, `bank_short`, `account_masked`, `recipient_key`, `kind` (account/card), `source` (manual/vietqr), `qr_amount`, `qr_memo` |
| **Indicator** | Dấu hiệu nhận diện: `account`, `phone` (E.164), `domain`, `url` |
| **VietQR** | QR chuyển khoản theo chuẩn EMVCo/NAPAS: TLV, tag 38 (GUID + BIN + STK), 54 (số tiền), 62 (memo), 63 (CRC-16/CCITT-FALSE). Chi tiết cần đối chiếu spec NAPAS [CHƯA KIỂM CHỨNG] |
| **Name inquiry (mock)** | Giả lập tra tên chủ tài khoản: `holder_name_masked`, `holder_type` (personal/...), `source: mock` |
| **Org Registry** | Bảng tổ chức chính thức: `org_id`, `name`, `aliases`, `domains`, `hotlines`, `official_account_keys`, `never_asks`, `source_url`, `verified_at` |
| **Group Trust** | Dữ liệu tin cậy của một nhóm Discord: `anchors` (người có vai trò), `declared_fund_accounts`, `payment_policy` (`declared_accounts_only` / `none`) |
| **`guild_hash`, `sender_hash`, `member_hash`, `reporter_hash`** | Hash có salt thay cho Discord ID. Không bao giờ lưu ID gốc |
| **`screening_id`** | ID ngắn hạn (`scr_...`) do `/v1/screen` sinh ra; trỏ tới claim context trong cache in-memory TTL 30 phút. Hết hạn thì bỏ qua + reason `screening_expired` |

## Evidence

| Thuật ngữ | Nghĩa |
| --- | --- |
| **Evidence source** | `mod_confirmed`, `user_report`, `public_warning`, `curated_feed`, `simulation` (người gọi khai) |
| **Evidence status** | `verified`, `unverified`, `simulation`, `retracted` (server tự đặt, không phải người gọi) |
| Quy tắc status | `mod_confirmed` hoặc `public_warning` có `source_url` → verified; `user_report` → unverified; `simulation` → simulation |
| **Dedup key** | `(indicator.key, scam_type)`. Merge cập nhật `last_seen`, `distinct_reporters`, chỉ được nâng status; `retracted` chỉ admin |
| **TTL** | verified 180 ngày; unverified 30 ngày (gia hạn khi có reporter mới); simulation không hết hạn nhưng có cờ |
| **`scam_type`** | `impersonation_org`, `impersonation_role`, `fake_fundraising`, `investment`, `job_task`, `phishing_link`, `loan`, `other` |

## Golden cases (contract §8)

`G1`, `G2` (green) · `R1`–`R4` (red) · `A1`–`A3` (amber) · `Y1` (grey) · `E1` (422). Chi tiết: SPEC.md §5.
