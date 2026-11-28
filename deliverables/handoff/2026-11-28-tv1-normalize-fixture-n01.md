# Handoff TV1 — fixture normalise N01 (T-13) · 2026-11-28

## Tóm tắt
- Fixture **N01**: một người nhận synthetic viết **6 cách** (4 nhập tay, 2 VietQR) phải ra **cùng một `recipient_key`**. Đây là bộ test chống lệch normaliser giữa TV1 (Guardian) và TV3 (SafeTransfer), đúng vấn đề số 2 của contract §1.
- Nằm ở `contracts/fixtures/normalize/n01_same_recipient_six_ways.json` trong REPO, nhánh `tv3/normalize-fixtures` (TV3 commit). Test phía TV3: `tests/contract/test_normalize_fixtures.py` (19 test pass).
- Fixture dùng **khoá HMAC giả công khai** (`hmac_key` trong file, có chữ `not-a-secret`). Không bao giờ dùng khoá này ở nơi khác, và không đưa `FW_HMAC_KEY` thật vào CI.

## Nội dung fixture

| Trường | Ý nghĩa |
| --- | --- |
| `hmac_key` | Khoá giả, dùng **nguyên chuỗi UTF-8**, không hex-decode (D-044) |
| `key_formula` | `rk_ + hex(HMAC_SHA256(utf8(hmac_key), 'acct:{bank_bin}:{normalized_account_no}'))[:32]` |
| `normalized_account_no` | STK sau chuẩn hoá (synthetic, dải `999000…`) |
| `expected` | `bank_bin`, `bank_short`, `account_masked`, `recipient_key` |
| `inputs[]` | 6 đầu vào: `kind` = `manual` (`bank`, `account_no`) hoặc `vietqr` (`payload`, `qr_amount`) |
| `must_differ[]` | 3 người nhận khác phải ra key **khác** (khác ngân hàng, đổi số cuối, thêm số 0 đầu) |

6 đầu vào: (a) STK thường + "VCB"; (b) có dấu cách, alias viết thường; (c) dấu chấm/gạch + tên đầy đủ "Vietcombank"; (d) BIN `970436`, có tab và NBSP (viết dạng ` ` trong JSON); (e) VietQR tĩnh; (f) VietQR tĩnh có `amount` 250000 (amount được trả về nhưng **không** nằm trong key).

## Cách chạy trong CI của TV1

Cách chạy hiện tại: **import từ REPO của TV3** (K01 chưa chốt cách đóng gói, xem bảng bên dưới).
```
git fetch && git switch tv3/normalize-fixtures
uv sync
uv run pytest tests/contract/test_normalize_fixtures.py -q
```
Đoạn mã tối thiểu để chạy fixture bằng chính hàm của TV1 (đã chạy thử, in `N01 ok: 6 inputs`):
```python
import json
from pathlib import Path

from fraudweave.common.normalize import normalize_manual_recipient
from fraudweave.common.vietqr.parse import parse_vietqr

case = json.loads(Path("contracts/fixtures/normalize/n01_same_recipient_six_ways.json").read_text(encoding="utf-8"))
key = case["hmac_key"].encode("utf-8")  # as-is, never hex-decoded
for item in case["inputs"]:
    if item["kind"] == "manual":
        got = normalize_manual_recipient(item["bank"], item["account_no"], key)
    else:
        got = parse_vietqr(item["payload"], key)
    assert got.recipient_key == case["expected"]["recipient_key"], item["id"]
    assert got.account_masked == case["expected"]["account_masked"], item["id"]
```
- Nếu bạn dùng `fraudweave.common` thì key phải khớp. Nếu bạn tự viết normaliser (không khuyến nghị, K01) thì fixture này sẽ cho biết lệch ở đầu vào nào qua trường `id`.
- Với STK tới từ tin nhắn (Guardian trích indicator), vẫn phải đi qua `normalize_account_number` rồi mới `recipient_key`. `recipient_key` không tự làm sạch đầu vào.

## Việc bạn cần chốt

| Kxx | Câu hỏi | Hạn |
| --- | --- | --- |
| K01 | Cách import `fraudweave.common` trong repo của bạn: cùng repo, submodule hay package riêng? Fixture N01 chạy được trong CI của bạn chưa? | họp T-06d, trước 30/11 |
| K01 | Bạn có cần thêm ca nào (ví dụ STK có chữ số Arabic/fullwidth → `invalid_recipient`, hay 5 chữ số) để Guardian bắt đúng lỗi? Báo TV3, ca mới sẽ thêm thành N02 | trước G2 (5/12) |

## Chưa có
- `contracts/fixtures/normalize/` mới có **một** case (N01). Chưa có fixture cho input lỗi.
- Chưa đóng gói thành package cài được; hiện chỉ import từ REPO.
- Ảnh QR (decode ảnh) chưa có: T-11/T-12 làm sau. Payload QR trong N01 là chuỗi đã decode.

## Đang là stub
Không có marker PENDING nào liên quan tới N01 (K01 là DRAFT-TV3, không phải PENDING).
