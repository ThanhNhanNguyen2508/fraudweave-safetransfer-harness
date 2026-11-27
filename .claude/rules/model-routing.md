# Rule: Phân vai model (D-030)

## Ai làm gì
| Việc | Model |
| --- | --- |
| Implement task theo PLAN (skill `implement-task`) | **Sonnet** |
| Review diff, contract audit, kiểm tra trước gate, lập kế hoạch, quyết định khi gặp điểm mơ hồ | **Opus** |
| Task rủi ro cao: **T-17** (policy), **T-07a** (đối chiếu spec VietQR/BIN), **T-24** (nhãn eval) | **Opus viết**, hoặc Sonnet viết rồi **Opus review kỹ** (đọc từng dòng, không chỉ lướt diff) |

## Agent và skill
- `.claude/agents/code-reviewer.md` và `contract-auditor.md` có `model: opus` trong frontmatter.
- Skill `implement-task` kết thúc bằng bước gọi `code-reviewer` review diff, **rồi mới** báo TV3.

## Agent không tự đổi model của phiên
- Không có cách (và không được cố) đổi model của phiên đang chạy. Chỉ TV3 đổi bằng `/model`.
- Nếu task cần Opus (bảng trên) mà phiên đang Sonnet: **dừng trước khi bắt đầu**, nhắc TV3: "Task này cần Opus, hãy chạy `/model` rồi báo tôi." Không tự làm tiếp bằng Sonnet.
- Gặp điểm mơ hồ khi đang chạy Sonnet: dừng và hỏi TV3 (hoặc đề nghị chuyển Opus), không tự quyết.

## Trạng thái kiểm chứng
- Trường `model` trong frontmatter subagent nhận alias (`sonnet`, `opus`, `haiku`, `fable`), model ID đầy đủ hoặc `inherit` [ĐÃ XÁC MINH qua tài liệu code.claude.com/docs/en/sub-agents, 26/11/2026; `claude --help` bản 2.1.332 cục bộ không mô tả frontmatter].
- Chưa chạy thử thực tế việc agent `code-reviewer` khởi chạy bằng Opus [CHƯA KIỂM CHỨNG]. Biến `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` nếu được đặt sẽ ghi đè trường `model` này.
