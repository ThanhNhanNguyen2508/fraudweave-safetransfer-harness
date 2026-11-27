# Handoff 2026-11-27: K04 thay đổi D-048 (TV1 + TV2)

> Từ TV3. Trạng thái K04: **DRAFT-TV3, bảng v2 proposed**, vẫn chờ TV1/TV2 duyệt (như handoff T-06c). Thay đổi này bổ sung cho bảng v2, không thay thế.
> Code: `src/fraudweave/safetransfer/policy.py`, `src/fraudweave/common/reasons.py` (T-17). `policy_version` giữ `st-policy-0.2` vì chưa phát hành.

## Thay đổi gì

1. **Miễn C3 khi khớp official.** STK khớp official account của **đúng** org được claim thì holder `personal` không bắn C3, kể cả khi role là org_staff/bank_staff/police. C2/C5/C6 và `registry_conflict` không đổi.
2. **Ghép cặp claim với loại tài khoản.** Claim org (resolve được hay không) chỉ được xác nhận bằng official account của org đó. Claim group chỉ được xác nhận bằng fund account đã khai báo của group. `match_declared` không xác nhận claim org, `match_official` không xác nhận claim group.

## Ca đổi kết quả

| Tình huống | Trước | Sau |
| --- | --- | --- |
| Claim org resolve, STK là official, holder `personal` | red C3 | **green** `matches_official_account` |
| Claim org resolve, STK là fund account của group, không phải official | green `matches_declared_account` | **red** C2 |
| Claim org **không** resolve, STK là fund account của group | green `matches_declared_account` | **amber** `org_not_in_registry` |
| Group `declared_accounts_only` + role teacher/treasurer/admin, STK chỉ khớp official của org được claim | green | **red** C2 |
| Như dòng 1, thêm verified evidence | red C3 + C5 | **amber** `registry_conflict` |
| Claim org + STK là fund account của group, thêm verified evidence | amber `registry_conflict` | **red** (C2 + C5 nếu org resolve, C5 nếu không) |
| Như dòng 4, thêm verified evidence | amber `registry_conflict` | **red** C2 + C5 |

Lưu ý dòng 4: giáo viên bảo chuyển học phí vào đúng tài khoản official của trường (có trong registry), nhưng nhóm đặt `declared_accounts_only` mà chưa khai tài khoản đó → **Đỏ**. Muốn Xanh thì group phải khai tài khoản này là fund account.

**12 golden fixture** trong `contracts/fixtures/transfer/` (G1, G2, R1–R4, A1–A4, Y1, E1) **không đổi** verdict, reason code hay fixture JSON. Reason codes, enum, models và `openapi.json` không đổi.

## Câu chữ (không đổi reason code)

- C2 chỉ bắn ở nhánh quỹ nhóm (ca R3, và dòng 4 ở trên) có câu riêng (service dùng `c2_uses_group_fund_wording()`): "Tài khoản này không khớp tài khoản quỹ mà nhóm đã khai báo." Cùng reason code `payee_not_official_account`.
- `registry_conflict`: "Tài khoản này khớp với tài khoản đã đăng ký hoặc đã khai báo, nhưng đã có báo cáo lừa đảo được xác minh. Hãy gọi kiểm tra qua kênh chính thức trước khi chuyển."
- `unverified_reports_on_official`: bỏ câu "đã chuyển cho người kiểm duyệt", vì service chưa chuyển báo cáo đi đâu.

## Ai cần làm gì

- **TV1:** nếu `engine.screen()` tự suy ra C2/C3 hoặc hiển thị trước verdict, áp dụng đúng 2 luật trên. Xác nhận đồng ý D-048 cùng lúc duyệt bảng v2 (K04).
- **TV2:** UI hiển thị `message_vi` từ response. Không suy verdict ở client. Không cần đổi mock (fixtures giữ nguyên). Xác nhận đồng ý D-048 cùng lúc duyệt bảng v2 (K04).
