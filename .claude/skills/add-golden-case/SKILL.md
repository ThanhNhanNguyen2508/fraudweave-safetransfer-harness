---
name: add-golden-case
description: Thêm một golden case cho POST /v1/transfer-risk (fixture JSON request + expected, seed synthetic cần thiết, và test tương ứng) vào REPO. Dùng khi cần thêm/sửa ca trong contracts/fixtures/transfer/ hoặc tái hiện một bug verdict.
---

# add-golden-case

## Đầu vào cần có (hỏi TV3 nếu thiếu)
- ID (theo SPEC §5, ví dụ `R2`; ca mới ngoài 11 ca gốc dùng tiền tố `X` + số, ví dụ `X01`).
- Seed data cần, request, expected verdict + reason codes.

## Bước
1. Kiểm tra ca chưa tồn tại trong `REPO/contracts/fixtures/transfer/` và expected khớp decision table (SPEC §4). Không khớp → dừng, báo TV3 (có thể là đổi policy, cần quy trình DRAFT-TV3).
2. Tạo `contracts/fixtures/transfer/<ID>_<slug>.json`:
   ```json
   {
     "id": "R2",
     "description": "personal holder, claim bank_staff of VCB-demo",
     "synthetic": true,
     "seed": ["registry:vcb_demo", "name_inquiry:account_d_personal"],
     "request": { "...": "TransferRiskRequest" },
     "expected": { "http_status": 200, "verdict": "red", "reason_codes": ["payee_not_official_account", "personal_holder_for_org_claim"] }
   }
   ```
   STK chỉ là số synthetic; không dùng số thật.
3. Thêm seed synthetic có cờ vào `data/seed/` nếu cần (idempotent).
4. Test `tests/golden/test_transfer_golden.py` tự nạp mọi fixture; nếu chưa có cơ chế đó thì thêm param cho case mới. Request phải validate bằng Pydantic models.
5. Chạy `uv run pytest tests/golden tests/contract` + ruff.
6. Ca thuộc 11 golden gốc đổi expected → coi là đổi contract DRAFT-TV3 (rule `contract-pending.md`).
7. Cập nhật `MAP.md`, session-log; tóm tắt: ID, verdict, file thêm, kết quả test.
