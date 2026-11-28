# Handoff TV1 + TV2 — `POST /v1/evidence` và quyền gửi evidence (K05 / K06) · 2026-11-28

## Tóm tắt
- TV3 đã làm xong endpoint ghi evidence. Nó **chưa commit** (nhánh `tv3/evidence-ingestion`) và **T-21 chưa nghiệm thu**: test cần PostgreSQL chưa chạy. K05 vẫn **DRAFT-TV3**, **K06 vẫn PENDING**: TV3 chưa chốt thay hai bạn, mọi điểm dưới đây là mặc định tạm của TV3 để hai bạn xác nhận hoặc đề xuất khác.
- Hình dạng request/response **không đổi** so với contract §5.3. Khác biệt là **quyền gửi** và vài giới hạn (bên dưới).

## Việc hai bạn cần biết trước

| # | Nội dung | Ai |
| --- | --- | --- |
| 1 | **MVP chỉ nhận indicator `recipient`** (tài khoản nhận tiền: nhập tay hoặc `vietqr_payload`). `phone` và `url` trả **400**. Đây là thu hẹp so với contract §5.3: chưa có dạng khoá HMAC cho phone/url và sẽ phải lưu giá trị thô. | TV1, TV2 |
| 2 | **Quyền gửi theo từng client (API key), mặc định từ chối.** Danh tính client lấy từ API key phía server, **không** lấy từ body. `user_report` (chỉ tạo `unverified`, không gây Đỏ) mở cho mọi key hợp lệ. `mod_confirmed`, `public_warning`, `curated_feed`, `simulation` cần TV3 cấp quyền rõ ràng cho key đó (biến `FW_EVIDENCE_GRANTS`, ví dụ `bot:mod_confirmed,loader:curated_feed+public_warning`). Gửi source chưa được cấp → **403** (`error.code = bad_request`, chưa có mã `forbidden` riêng; xem câu hỏi 3). | TV1, TV2 |
| 3 | **`mod_confirmed`: Discord Bot (TV2) phải tự xác minh người bấm là moderator trước khi gửi.** Server chỉ biết "key của bot được phép gửi `mod_confirmed`", không biết ai bấm. Nếu bot gửi mà chưa kiểm quyền thì bất kỳ thành viên nào cũng tạo được evidence verified (dẫn tới verdict Đỏ cho một tài khoản). | TV2 |
| 4 | **`curated_feed` = evidence verified do đội FraudWeave tự thu thập và kiểm tra**. Chỉ client nội bộ được cấp quyền (ví dụ script nạp dữ liệu công khai của TV1) mới gửi được. Dữ liệu ngoài chưa kiểm duyệt không được gửi dưới source này. | TV1 |
| 5 | **`reporter_hash`**: dạng `r_...` (chữ, số, `_`, `-`; tối đa 64 ký tự), là hash có salt, không phải Discord ID. Sai dạng → 400. Không gửi `reporter_hash` thì `distinct_reporters` = 0 và báo cáo đó không gia hạn gì. | TV2 |
| 6 | **`source_url`** (bắt buộc với `public_warning`): chỉ nhận link `http(s)` thường. **Bị từ chối (400)**: có user/password, có query (`?`), có fragment (`#`), có khoảng trắng hoặc ký tự điều khiển, dài hơn 2048 ký tự, thiếu host. Chỉ nhận ký tự ASCII in được (không có ký tự điều khiển, RTL override, zero-width, host Unicode), và không có `\\` hay `%` trong host. Server **không sửa** URL. Một `source_url` đến sau chỉ được điền vào dòng chưa có nguồn nếu report đó đáng tin ít nhất bằng dòng (nên `user_report` không gắn được link vào evidence verified). **Giới hạn:** path vẫn có thể chứa định danh (ví dụ id hồ sơ) và việc phân tích URL không phát hiện hết dữ liệu cá nhân, nên chỉ gửi trang cảnh báo công khai. | TV1, TV2 |
| 7 | **`simulation`**: chỉ nhận với số tài khoản nhập tay trong dải synthetic `999000…`, không nhận `vietqr_payload`. Chỉ tính khi `mode=demo`. | TV2 |

**Lưu ý rate limit:** `POST /v1/evidence` dùng **chung** bucket 60 request/phút của mỗi key với `transfer-risk`; bot nên dùng key riêng cho việc gửi evidence nếu lo bị tranh ngân sách. `user_report` mở cho mọi key hợp lệ và `reporter_hash` do người gọi tự đặt, nên **đừng nhúng key vào frontend công khai** (T-23/K12).

## Quy tắc ghi và merge (chốt tạm, D-057)
- Status do server đặt, **không** nhận từ body: `mod_confirmed`, `public_warning` (có `source_url`), `curated_feed` → `verified`; `user_report` → `unverified`; `simulation` → `simulation`.
- Trùng `(tài khoản, scam_type)` thì merge vào một dòng: **201** nếu tạo mới, **200** nếu merge. Merge chỉ **nâng** status (unverified → verified), không hạ.
- TTL: verified 180 ngày, unverified 30 ngày (reporter mới gia hạn), simulation không hết hạn nhưng có cờ.
- **Gia hạn `mod_confirmed`:** chỉ khi xác nhận đến từ một `reporter_hash` **chưa từng thấy** trên dòng đó. Gửi lại cùng reporter hoặc không có `reporter_hash` chỉ cập nhật `last_seen` (có thể là retry). Contract hiện chưa có id xác nhận để phân biệt xác nhận lại thật với retry: **câu hỏi cho T-06d**.
- Dòng đã **hết hạn** được coi là một lần ghi mới (reset status, reporters, nguồn). Dòng **`retracted`** không bị report mới đánh thức hay thay đổi (chỉ admin).
- Simulation và dữ liệu thật trên cùng khoá → **400**, không trộn.

## Mã lỗi
`401` thiếu/sai key · `429` rate limit (chung bucket với `transfer-risk`, 60/phút) · `403` source chưa được cấp quyền · `400` body sai, indicator phone/url, `source_url` không hợp lệ, `reporter_hash` sai dạng, trộn simulation/real · `422` số tài khoản/ngân hàng không dùng được (giống `transfer-risk`) · `503` lỗi lưu trữ. Thứ tự: đọc body → key (401) → rate limit (429) → quyền source (403) → kiểm tra còn lại. Lưu ý: body JSON hỏng mà không có key trả **400** (FastAPI đọc body trước khi kiểm key), giống hệt `transfer-risk`.

## Câu hỏi cần chốt (họp T-06d, trước 30/11)
1. **K06 (TV2):** bot gửi `mod_confirmed` bằng key riêng nào, và bot xác minh quyền moderator ra sao? Nút "Báo cáo" gửi `user_report` có `reporter_hash` như thế nào (cùng salt `sender_hash`?).
2. **K06 (TV1):** loader dữ liệu công khai dùng `curated_feed` hay `public_warning`? Có key riêng không?
3. **K05:** thêm mã lỗi `forbidden` (403) vào envelope §7? Hiện tạm dùng `bad_request`.
4. **K05/K06:** thêm `confirmation_id` (idempotency key) để phân biệt xác nhận lại thật của moderator với retry? Hiện chỉ là giải pháp tạm: chỉ `reporter_hash` mới khác mới gia hạn verified (TV3 đã chấp nhận tạm, D-058).
5. **K05:** có cần `phone`/`url` indicator trong MVP không? Nếu có, TV3 cần chốt dạng khoá trước.
6. **K06:** ai được `retract` evidence (admin TV3 hay mod server)? Endpoint retract chưa có.

## Chưa kiểm chứng
- Hành vi cần PostgreSQL (dedup, race, merge, hết hạn) đã có test nhưng **chưa chạy trên PostgreSQL thật** [CHƯA KIỂM CHỨNG] cho tới khi TV3 chạy full regression.
