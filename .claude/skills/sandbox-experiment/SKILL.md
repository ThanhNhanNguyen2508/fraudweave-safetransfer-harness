---
name: sandbox-experiment
description: Chạy một thử nghiệm nháp (khám phá thư viện, benchmark thử, decode QR thử, notebook) trong SANDBOX Test-for-agent, không bao giờ ghi vào REPO. Dùng khi cần thử trước khi viết code chính thức, hoặc khi kết quả chỉ là tạm.
---

# sandbox-experiment

SANDBOX = `C:\Users\Dell\Downloads\MomoTechDay\Implements\Test-for-agent`.

## Bước
1. Tạo thư mục `SANDBOX/YYYY-MM-DD-<slug>/` (slug tiếng Anh, ngắn).
2. Tạo `README.md` đúng 3 dòng:
   ```
   Goal: <muốn biết điều gì>
   Result: <kết quả, hoặc "pending">
   Next: <đưa vào REPO qua T-xx / bỏ>
   ```
3. Viết script/nháp trong thư mục đó. Có thể **đọc** code REPO (thêm `src` vào `PYTHONPATH` hoặc dùng `uv run --project <REPO>`), nhưng **không ghi** gì vào REPO.
4. Không dùng `.env` thật hay dữ liệu thật; dùng key giả và dữ liệu synthetic.
5. Chạy, ghi kết quả vào dòng `Result:`. Số liệu phải là output thật, ghi kèm lệnh đã chạy.
6. Kết quả đáng giữ → đề xuất task T-xx để viết lại sạch vào REPO (kèm test). Không copy nguyên file nháp.
7. Cập nhật `MAP.md` (mục SANDBOX, một dòng cho thư mục thử nghiệm) và session-log.

Nhớ: xoá toàn bộ SANDBOX thì REPO vẫn phải chạy và CI vẫn pass.
