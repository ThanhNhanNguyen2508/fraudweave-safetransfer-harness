---
name: contract-audit
description: Quét mọi marker PENDING-NEED-CONFIRM-CONTRACT trong REPO và đối chiếu với CONTRACT_STATUS.md, báo phần lệch. Dùng khi "chốt phiên", trước mỗi gate (G1/G2/G3), hoặc sau khi đổi contract.
---

# contract-audit

Chỉ đọc. Không sửa code. Chỉ được sửa `CONTRACT_STATUS.md` nếu TV3 đồng ý sau khi xem báo cáo.

## Bước

1. Grep trong REPO (bỏ `.venv`, `.git`):
   - Marker đúng: `PENDING-NEED-CONFIRM-CONTRACT\(K\d{2}\): .+`
   - Marker sai định dạng: dòng chứa `PENDING-NEED-CONFIRM-CONTRACT` nhưng không khớp mẫu trên.
2. Gom marker theo Kxx (file:line + mô tả).
3. Đối chiếu với `CONTRACT_STATUS.md`:
   - Kxx có marker nhưng trạng thái **không phải** PENDING → lệch (có thể quên gỡ marker hoặc quên cập nhật bảng).
   - Kxx PENDING nhưng **không có** marker và cột "Stub/adapter" ghi có code → lệch.
   - File trong cột "Stub/adapter" không tồn tại trong REPO → lệch.
   - Kxx không tồn tại (ngoài K01–K12) → lệch.
4. Đối chiếu nhanh `REPO/contracts/CONTRACT_STATUS.md` (bản sao) với bản HARNESS: trạng thái phải giống nhau.
5. Kiểm tra fixtures: mọi file trong `contracts/fixtures/transfer/` có đủ request + expected; đủ 11 golden case của SPEC §5.

## Báo cáo (trả lời TV3)

| Kxx | Trạng thái bảng | Marker trong code (file:line) | Lệch? | Đề xuất |
| --- | --- | --- | --- | --- |

Cuối báo cáo: số marker, số lệch, danh sách câu hỏi cần hỏi TV1/TV2.
