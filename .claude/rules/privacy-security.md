# Rule: privacy và bảo mật

## Không bao giờ lưu, log hay gửi ra ngoài
- Nội dung tin nhắn (`message.text`), `transfer_memo`.
- STK thô (chỉ dùng trong bộ nhớ để tính key), Discord user/guild ID.
- Số tiền chính xác (chỉ `amount_band`; `qr_amount` chỉ trả về trong response, không lưu).

## Chỉ được lưu
- `recipient_key` (HMAC-SHA256 với `FW_HMAC_KEY`), `account_masked` (4 số cuối), `amount_band`.
- Hash có salt: `guild_hash`, `sender_hash`, `member_hash`, `reporter_hash`.
- Audit log: `request_id`, `recipient_key`, verdict, reason codes, `policy_version`, latency. Không có định danh thô.

## Secrets
- `FW_HMAC_KEY`, `DATABASE_URL`, `API_KEYS` chỉ nằm trong `REPO/.env` (không commit). `.env.example` chỉ chứa placeholder.
- Agent **không đọc, không in** `.env` hay giá trị secret ra log, chat, test output hay file khác. Test dùng key giả trong fixture/conftest.
- `.env.example` (và `.sample` / `.template` / `.dist`) là template **không chứa secret**: agent được tạo, sửa, đọc. Hook toàn cục `gsd-secret-read-guard` chặn mọi lệnh đọc `.env` / `.env.<suffix>` khác. Không tìm cách lách hook (D-015).
- Commit dùng email noreply của GitHub (D-016), không dùng email cá nhân.
- Không hard-code secret trong code; không rotate `FW_HMAC_KEY` trong thời gian thi (sẽ phải tính lại mọi key).
- Supabase: bật RLS, chỉ backend service role được đọc/ghi.

## Dữ liệu
- Dữ liệu demo **luôn synthetic và có cờ** (`source = simulation` hoặc cờ synthetic trong seed). Không dùng STK/tên người thật.
- Dữ liệu tổ chức thật phải có `source_url` lấy từ website chính thức của tổ chức.
- Simulation evidence chỉ được tính khi `mode=demo`.

## An toàn verdict
- Lỗi 5xx/timeout **không bao giờ** trả Xanh; client hiển thị "Không kiểm tra được".
- Không dùng chữ "an toàn" trong copy của Xanh.
