# Handoff TV2 — `POST /v1/qr/decode`, xác nhận định dạng upload (K02) · 2026-11-28

## Tóm tắt
- TV3 đã làm xong endpoint đọc QR từ ảnh. Nó **chưa commit**, nằm ở nhánh `tv3/vietqr-decode` (TV3 commit). OpenAPI mới ở `contracts/openapi.json`.
- Định dạng upload là **mặc định TV3 chọn**: `multipart/form-data`, đúng một file tên `image`. K02 vẫn **DRAFT-TV3**: bạn xác nhận hoặc đề xuất khác thì TV3 đổi theo.
- Brief §5 đã chuyển việc decode ảnh từ TV2 sang TV3, nên Discord bot **không cần** tự decode ảnh nếu dùng endpoint này.
- Ảnh được đọc trong **process con có giới hạn thời gian 3 giây** (chống ảnh đối nghịch), nên có thêm mã lỗi 503, giới hạn **1 lần đọc đồng thời cho mỗi client** (429) và quy tắc kích thước QR bên dưới.

## Endpoint / API

| Mục | Giá trị |
| --- | --- |
| Method, path | `POST /v1/qr/decode` |
| Header bắt buộc | `X-API-Key`, và `Content-Length` (xem "Lưu ý khi gọi") |
| Header tuỳ chọn | `X-Request-Id` (server trả lại) |
| Body | `multipart/form-data`, **một** phần file tên `image`, **không** có trường nào khác |
| Định dạng ảnh | PNG hoặc JPEG, nhận biết theo nội dung file, không theo `Content-Type` |
| Giới hạn | file ≤ **5 MiB**, ảnh ≤ **16.000.000 pixel** (ví dụ 4000 × 4000) |
| Rate limit | 60 request/phút cho mỗi key, **bucket riêng** với `transfer-risk` |
| Đồng thời | **tối đa 1 request QR decode cho mỗi API key**, tính **từ lúc request được nhận** (gồm cả lúc upload tối đa 15 giây và lúc chờ hàng đợi); request thứ hai cùng key lúc đó nhận 429 ngay |
| Thời gian xử lý | quan sát: ấm **0,05 đến 0,1 giây**, nguội (worker mới) **~1 giây**. Giới hạn tối đa: đọc ảnh 3 giây, chờ hàng đợi 2 giây, khởi động worker 30 giây (van cuối), nên một request lành mạnh vẫn trong khoảng 4 giây |
| Thời gian gửi | cả upload phải đến trong **15 giây**, quá hạn trả 400 |

Thứ tự xử lý: kiểm key (401) → rate limit theo phút (429) → giới hạn 1 lần đọc mỗi client (429) → kiểm kích thước và dạng form (400) → đọc ảnh (422/503) → kiểm VietQR (422).

| HTTP | `error.code` | Khi nào | UI nên làm gì |
| --- | --- | --- | --- |
| 200 | — | Đọc được một mã VietQR hợp lệ | Dùng `recipient` để hiển thị, gửi `vietqr_payload` sang `transfer-risk` |
| 400 | `bad_request` | Sai dạng form: không có file, sai tên trường, trường thừa, 2 file, body JSON, thiếu `Content-Length`, có `Transfer-Encoding`. Hai ca khác: **upload quá 15 giây** hoặc ngắt kết nối giữa chừng (do mạng, nên thử lại) | Các ca sai dạng: lỗi client, sửa code gọi. Ca quá 15 giây hoặc ngắt kết nối: thử lại |
| 401 | `invalid_api_key` | Thiếu hoặc sai key | Lỗi cấu hình |
| 422 | `invalid_qr` | Đã **xác định** ảnh hoặc mã không dùng được: quá lớn, quá nhiều pixel, hỏng, sai định dạng, không có QR, QR không phải VietQR, CRC sai, ảnh có hai mã khác nhau | Bảo người dùng chụp hoặc tải ảnh khác (rõ hơn, cắt sát mã) |
| 422 | `unknown_bank` | Mã VietQR hợp lệ nhưng ngân hàng ngoài 10 ngân hàng hỗ trợ | Báo ngân hàng chưa hỗ trợ |
| 422 | `invalid_recipient` | Số tài khoản trong mã không hợp lệ | Bảo người dùng nhập tay |
| 429 | `rate_limited` | Vượt 60 request/phút, **hoặc** client này đã có một lần đọc ảnh đang chạy. Có `Retry-After` (giây; 3 với trường hợp đang đọc) | Chờ rồi thử lại, không hiện lỗi về ảnh |
| **503** | `engine_unavailable` | Bộ đọc ảnh bận (hàng đợi đầy hoặc chờ quá 2 giây, kèm `Retry-After: 2`), **đọc ảnh quá 3 giây**, worker không khởi động được hoặc bị lỗi | Hiện "Không kiểm tra được, thử lại sau". **Không** diễn giải thành "QR không hợp lệ" hay "tài khoản nguy hiểm", và **không** coi là Xanh |
| 500 | `internal_error` | Lỗi server hoặc thiếu cấu hình | Hiện "Không kiểm tra được" |

Khác biệt cần nhớ: **422 là đã xác định lỗi của ảnh hoặc mã** (đổi ảnh có thể giúp). **503 chưa nói gì về ảnh**: có thể do hệ thống bận hoặc ảnh quá nặng để đọc trong 3 giây, nên UI chỉ báo "không kiểm tra được" và cho thử lại hoặc nhập tay số tài khoản.

## Ví dụ (chạy thật trên code, dữ liệu synthetic)

Request, dùng ảnh `g2_class_fund.png` sinh bởi `scripts/gen_demo_qr.py` (xem "Cách chạy"):
```
POST /v1/qr/decode
X-API-Key: <key>
Content-Type: multipart/form-data; boundary=...
(một phần file: name="image"; filename="qr.png")
```
Response 200 (có header `Cache-Control: no-store`, `X-Request-Id`):
```json
{
  "vietqr_payload": "00020101021238570010A00000072701270006970422011399900000000020208QRIBFTTA530370454062000005802VN62140810Quy lop 5A63040F0F",
  "recipient": {
    "bank_bin": "970422",
    "bank_short": "MB",
    "account_masked": "*********0002",
    "recipient_key": "rk_0ae4dadcbc5646f2dd55830cf17db2cc",
    "kind": "account",
    "source": "vietqr",
    "qr_amount": 200000,
    "qr_memo": "Quy lop 5A"
  }
}
```
(`recipient_key` phụ thuộc khoá HMAC của từng môi trường, đây chỉ là giá trị với khoá thử.)

Response 422 (ảnh không có QR):
```json
{"error": {"code": "invalid_qr", "message": "No readable QR code was found in the image", "request_id": "req_example"}}
```
Response 429: header `Retry-After: 60`, body `{"error": {"code": "rate_limited", "message": "Rate limit exceeded", "request_id": "..."}}`.
Response 503 quá tải: header `Retry-After: 2`, body `{"error": {"code": "engine_unavailable", "message": "Image decoding is busy, try again shortly", ...}}`. Đọc ảnh quá 3 giây cũng là 503 (`"The image could not be read in time"`, không có `Retry-After`).
Response 429 khi client đang có một lần đọc: header `Retry-After: 3`, body `{"error": {"code": "rate_limited", "message": "A QR decode for this client is already in progress", ...}}`.

Gửi tiếp sang kiểm tra người nhận: lấy `vietqr_payload` từ response, bỏ vào `POST /v1/transfer-risk` như `{"recipient": {"vietqr_payload": "<chuỗi đó>"}, "amount_band": "1m_10m"}`. Test của TV3 đã kiểm bước này với dịch vụ giả.

## Lưu ý khi gọi (quan trọng cho bot và web)
- **`vietqr_payload` chứa số tài khoản đầy đủ.** Đừng lưu hay log nó. Server không lưu, không log, response có `Cache-Control: no-store`. Đây là ngoại lệ privacy TV3 đã duyệt, chỉ cho endpoint này.
- **Ảnh lớn bị thu nhỏ về cạnh dài tối đa 1500 px trước khi đọc.** Ảnh nhỏ hơn giữ nguyên. Hệ quả: nếu ảnh lớn (ví dụ ảnh chụp 4000 px) mà mã QR chiếm **dưới khoảng 12% cạnh dài**, mã có thể không đọc được (đo trên ảnh sạch: 11% đọc được, 9% thất thường). Hãy gợi ý người dùng cắt sát mã hoặc chụp gần hơn.
- **Phần file phải có `filename` khác rỗng.** Thiếu tên hoặc tên rỗng đều trả 400.
- **Phải gửi `Content-Length`, không dùng chunked.** Server từ chối `Transfer-Encoding`. Thư viện HTTP thường tự đặt `Content-Length` khi file nằm trong bộ nhớ; nếu bạn stream file thì cần đọc hết trước. Bạn cho TV3 biết nếu thư viện của bạn không làm được.
- Ảnh có **hai mã QR khác nhau** bị từ chối (422), kể cả khi mã thứ hai không đọc được.
- Ảnh PNG nền trong suốt được ghép lên nền trắng, nên vẫn đọc được.
- **Đừng đặt API key dịch vụ vào mã frontend chạy trên trình duyệt** khi triển khai công khai: ai mở trang cũng lấy được key. Việc này TV3 ghi vào câu hỏi mở của T-23/K12 (cần BFF/proxy hoặc cách khác), chưa làm.

## Cách chạy local
```
uv sync
uv run python scripts/gen_demo_qr.py        # ghi 3 ảnh demo vào out/demo_qr (thư mục git-ignored)
uv run uvicorn fraudweave.api.main:app --reload
curl -X POST http://127.0.0.1:8000/v1/qr/decode -H "X-API-Key: <key trong .env>" -F "image=@out/demo_qr/g2_class_fund.png"
```
Cần `.env` có `FW_HMAC_KEY` và `API_KEYS` (xem `.env.example`), `DATABASE_URL` cũng phải có vì dùng chung cấu hình app. Ba ảnh demo dùng tài khoản synthetic của golden G1, R1, G2. Lần gọi đầu sau khi khởi động mất khoảng 1 giây (khởi động worker), các lần sau khoảng 0,05 giây.

## Việc bạn cần chốt

| Kxx | Câu hỏi | Hạn |
| --- | --- | --- |
| K02 | Multipart một trường `image` có dùng được từ Discord bot và simulator không? Cần thêm kiểu khác (base64 trong JSON, URL ảnh)? | trước 1/12 |
| K02 | Giới hạn 5 MiB, 16 triệu pixel và việc thu nhỏ về 1500 px có phù hợp với ảnh người dùng gửi qua Discord/web? Thư viện của bạn có tự gửi `Content-Length` không? | trước 1/12 |
| K02 | UI hiển thị gì cho từng mã lỗi 422, 429 và 503 ở trên (phân biệt "đổi ảnh" với "thử lại sau"; 503 không được hiện thành QR không hợp lệ) | trước G2 (5/12) |
| K02 | Bot có thể xếp hàng theo Discord user và dùng chung một lần đọc đồng thời không? Nếu cần nhiều hơn (nhiều key, hoặc giới hạn cao hơn), nói với TV3 | trước G2 (5/12) |
| K03 | 60 request/phút cho `qr/decode` (bucket riêng) có đủ không | họp T-06d, trước 30/11 |

## Chưa có / còn mở
- Tương thích với ảnh chụp thật từ app ngân hàng [CHƯA KIỂM CHỨNG]: chỉ mới thử với ảnh sinh bằng `scripts/gen_demo_qr.py` và biến thể (xoay, JPEG nén, thu nhỏ, nền trong suốt, nhiễu).
- OpenCV có thể không đọc được mã có chữ có dấu trong memo nếu mã được tạo ở chế độ trộn số + byte [CHƯA KIỂM CHỨNG trên mã của app ngân hàng thật]. Khi đó trả 422 `invalid_qr`.
- Bộ đọc ảnh chạy trong process con để chống làm cạn tài nguyên; đó **không phải security sandbox**.
- **Bot dùng một key cho mọi người dùng Discord, nên các người dùng chia nhau đúng một lần đọc đồng thời.** Khi hai người quét cùng lúc, người thứ hai nhận 429 (`Retry-After: 3`). **Mỗi 429 do giới hạn đồng thời vẫn tính vào 60 request/phút**, nên bắn song song rồi thử lại có thể khoá cả bot tới 60 giây: hãy tự giữ một semaphore 1 (hoặc hàng đợi) ở phía bot thay vì dựa vào 429, và tải xong file ảnh từ Discord vào bộ nhớ **rồi mới** gọi API (chỗ được giữ từ lúc API nhận request). Giới hạn theo key chặn được một key chiếm hết bộ đọc ảnh, nhưng **không** đảm bảo công bằng giữa các người dùng: bạn cần **rate limit và xếp hàng theo từng Discord user ở phía bot** (ví dụ mỗi người một lượt đang chờ), và thử lại sau `Retry-After` giây khi nhận 429.
- Không có hạn mức tổng số kết nối mở (mỗi client tối đa 1 upload và hạn 15 giây mỗi upload); worker kế thừa môi trường của API: thuộc lớp triển khai (K12).

## Đang là stub
Không có marker PENDING nào ở endpoint này (K02 là DRAFT-TV3, không phải PENDING). Các marker liên quan khác: K07 (`screening_id`), K06 (evidence), không đổi.
