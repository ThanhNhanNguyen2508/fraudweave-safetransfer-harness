# Handoff cho TV1 — `recipient_key`, chuẩn hoá STK, bảng ngân hàng (27/11)

## Tóm tắt
- Đã sẵn sàng trong `fraudweave.common` (= `fw_common`): chuẩn hoá STK, `recipient_key` HMAC, che số, bảng 10 ngân hàng, CRC/TLV cho VietQR.
- Nguồn: repo `FraudWaveAI-Momo`, nhánh `tv3/vietqr-banks-tlv` (T-07, T-08, T-09). Test dùng key giả, không cần `.env` thật.
- **Chưa có:** parser VietQR đầy đủ (T-10), fixture normalise 6 cách (T-13). Hàm dưới đây chỉ dành cho nhập tay (`bank` + `account_no`).

## Cách dùng (K01, bắt buộc để key khớp với TV3)
| Việc | Hàm | Module |
| --- | --- | --- |
| Đọc khoá HMAC từ môi trường | `hmac_key_from_env()` | `common/hashing.py` |
| Chuẩn hoá STK | `normalize_account_number(raw)` | `common/normalize.py` |
| Ngân hàng → BIN | `resolve_bank(text)`, `bank_by_bin(bin)` | `common/banks.py` |
| Tính key | `recipient_key(bank_bin, account_no, hmac_key)` | `common/hashing.py` |
| Che số | `mask_account(account_no)` | `common/normalize.py` |
| Gộp tất cả thành `NormalizedRecipient` | `normalize_manual_recipient(bank, account_no, hmac_key)` | `common/normalize.py` |

Thứ tự gọi nếu không dùng hàm gộp: `resolve_bank` → `normalize_account_number` → `recipient_key`.

## Quy tắc TV1 phải theo (D-044, D-045)
- **Không tự viết lại normaliser hay công thức key.** `recipient_key = "rk_" + hex(HMAC_SHA256(FW_HMAC_KEY, f"acct:{bin}:{account_no}"))[:32]`.
- **Khoá HMAC là UTF-8 của chuỗi `FW_HMAC_KEY` nguyên dạng, không giải mã hex.** `.env.example` đã sửa câu chữ cho khớp. Hãy gọi `hmac_key_from_env()`; tự hex-decode sẽ làm mọi key lệch. Khoá thiếu hoặc ngắn hơn 32 ký tự → `InvalidHmacKeyError`.
- **`recipient_key()` không tự làm sạch đầu vào.** STK phải qua `normalize_account_number()` trước; nếu không sẽ raise `ValueError` thường (không phải `InvalidRecipientError`).
- Chuẩn hoá STK: bỏ khoảng trắng (theo `str.isspace()`), `.` và `-`; còn lại phải là 6–19 chữ số ASCII. Giữ số 0 đầu (`0071…` khác `71…`). Chữ số Arabic/fullwidth, `+84`, ký tự zero-width → lỗi.
- Ngân hàng: khớp **chính xác** sau chuẩn hoá (chữ thường, bỏ dấu, gộp khoảng trắng), không khớp mờ. Chỉ 10 ngân hàng (VCB, ICB, BIDV, VBA, TCB, MB, ACB, VPB, STB, TPB); BIN khác → `UnknownBankError`.

## Lỗi và cách map sang API
| Exception | Mã API | HTTP |
| --- | --- | --- |
| `InvalidRecipientError` | `invalid_recipient` | 422 |
| `UnknownBankError` | `unknown_bank` | 422 |
| `InvalidHmacKeyError` | lỗi cấu hình, không phải lỗi người dùng | 5xx, không bao giờ trả Xanh |

Việc map sang response do T-19 làm ở phía TV3. Thông báo lỗi không chứa STK hay key.

## Ví dụ kiểm tra (khoá giả, chỉ để test hồi quy)
Với khoá `regression-vector-key-not-a-secret-01`:

| BIN | STK (đã chuẩn hoá) | `recipient_key` |
| --- | --- | --- |
| 970436 | 0071001234567 | `rk_606ae6887dcd18564c3c0fcddc629dcf` |
| 970436 | 71001234567 | `rk_d6cf32655f8070610a027a6aec0e6bf6` |
| 970415 | 0071001234567 | `rk_4af02e14a655a0a67d45ff270124bb67` |
| 970422 | 123456 | `rk_8c54f164b1c9b38c3fab3f51f62c67ad` |

TV1 có thể chạy 4 dòng này trong CI của mình để chắc chắn key khớp. Che số: `1234567890` → `******7890`; `0071001234567` → `*********4567`.

## Cách chạy local
```
git fetch && git switch tv3/vietqr-banks-tlv
uv sync
uv run pytest tests/unit
```

## Việc TV1 cần chốt
| Kxx | Câu hỏi | Hạn |
| --- | --- | --- |
| K01 | Xác nhận import `fraudweave.common` (không tự viết lại), và cách đóng gói (cùng repo hay package riêng) | Trước 30/11 (T-06d) |
| K01 | Đồng ý hai điểm: khoá HMAC dùng nguyên chuỗi UTF-8, STK phải chuẩn hoá trước khi hash | Trước 30/11 |

## Đang là stub
- Không có marker PENDING mới ở T-07/T-08/T-09. K07 và K06 vẫn như trong ghi chú bàn giao 27/11 trước.

## Giới hạn đã biết
- Chưa giới hạn độ dài `account_no` ở tầng API (T-19 sẽ chặn); đừng gửi chuỗi quá dài.
- Chỉ hỗ trợ chuyển vào tài khoản (`QRIBFTTA`); QR chuyển vào thẻ ngoài phạm vi MVP.
