@AGENTS.md

# CLAUDE.md — SafeTransfer harness (TV3)

## Tôi là ai
- **TV3 (Rykanny)**, Backend & Fraud Intelligence Engineer của nhóm FraudWeave, MoMo Tech Contest 2026.
- Sở hữu end-to-end **SafeTransfer** và phần dữ liệu/backend dùng chung: `fw_common`, VietQR, Org Registry, Group Trust, Evidence Store, FastAPI app shell, eval, tích hợp E2E.
- Chi tiết ownership: @context/team-and-ownership.md

## Dự án làm gì
- **SafeTransfer** cảnh báo **trước khi chuyển khoản**: nhập STK hoặc quét VietQR → verdict **Đỏ / Vàng / Xám / Xanh** kèm bằng chứng và reason codes.
- Giá trị riêng (không chỉ là blacklist): **claim–payee mismatch**. Người nhận tự xưng là X nhưng tài khoản không khớp tài khoản chính thức/đã khai báo của X (C2), là tài khoản cá nhân (C3), đã có evidence verified (C5), hoặc X "không bao giờ yêu cầu" việc này (C6).
- Mục tiêu: ≥ 95% verdict đúng trên eval set ≥ 150 ca, **0 Xanh sai**, `POST /v1/transfer-risk` p95 ≤ 150 ms, chạy end-to-end độc lập với TV1/TV2.
- Gate: **G1 29/11 · G2 5/12 · G3 9/12 · TechDay 11/12/2026.**

## Đọc ở đâu
| Cần gì | Mở file |
| --- | --- |
| Bản đồ 3 thư mục, file nào ở đâu | @MAP.md |
| Phạm vi, mục tiêu, decision table, golden cases, Done theo gate | `SPEC.md` |
| Task T-xx, trạng thái, phụ thuộc | `PLAN.md` |
| Trạng thái K01–K12 (nơi duy nhất) | `CONTRACT_STATUS.md` |
| Thuật ngữ (Claim, C1–C9, verdict, recipient_key...) | `context/glossary.md` |
| Quyết định đã chốt và lý do | `context/decisions.md` |
| Nguồn gốc (chỉ đọc) | `sources/00-project-brief.md`, `sources/contract-v0.1.md`, `sources/pipeline-revised.md` |
| Lịch sử phiên | `deliverables/session-log.md` (đọc 10 dòng cuối) |

## Cách tôi làm việc
- **Build-first, integrate-later:** làm được nhiều nhất có thể; phần chưa chốt dùng stub/adapter + marker, không chờ.
- **Contract-first:** Pydantic models là nguồn sự thật; OpenAPI và fixtures sinh từ đó.
- **Evidence over assumptions:** dữ liệu demo synthetic và có cờ; dữ liệu tổ chức thật phải có `source_url`.
- **Deterministic policy:** verdict từ decision table, không phải điểm ML.
- Mỗi task đi qua skill `implement-task`; trước mỗi gate chạy `contract-audit` hoặc agent `contract-auditor`.
- Thử nghiệm nháp vào SANDBOX qua skill `sandbox-experiment`, không bao giờ vào REPO.
- Thay đổi xong: cho tôi xem diff + kết quả test, **chưa commit** cho tới khi tôi duyệt.
- Phiên song song: mỗi phiên một worktree ngoài REPO (`...\Implements\worktrees\`), theo @context/git-workflow.md.
- **Chốt phiên luôn do Claude Code làm** bằng skill `close-session`: nhập ghi chú của Codex trong `deliverables/codex-inbox/`, cập nhật harness, cho tôi xem `git status` và gợi ý commit message. **Tôi tự commit/push.**

## Quy tắc bắt buộc (nạp tự động)
@.claude/rules/folder-placement.md
@.claude/rules/contract-pending.md
@.claude/rules/privacy-security.md
@.claude/rules/coding-standards.md
@.claude/rules/definition-of-done.md
@.claude/rules/git-permissions.md

## Nhắc nhanh những điều không được làm
- Không viết code sản phẩm trong HARNESS. Không sửa `sources/`.
- Không tự chốt contract PENDING thay TV1/TV2.
- Không trả Xanh khi lỗi/timeout; không dùng chữ "an toàn" cho Xanh.
- Không bao giờ chạy `git add` / `commit` / `push` hay `gh`; không xoá hàng loạt.
- Không bịa số liệu, nguồn, kết quả. Luôn gắn [ĐÃ XÁC MINH] / [GIẢ ĐỊNH] / [CHƯA KIỂM CHỨNG] cho khẳng định quan trọng.
