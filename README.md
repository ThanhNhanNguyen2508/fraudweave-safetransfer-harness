# Agent Workspace — harness 3 tầng cho SafeTransfer (FraudWeave, TV3)

Đây là **harness** (bộ não và quy trình) cho Claude Code và Codex, không phải code sản phẩm. Code: REPO `..\Source code-SafeTransfer\FraudWaveAI-Momo`. Nháp: SANDBOX `..\Test-for-agent`. Phiên song song: `..\worktrees\`.
- **Tầng 1 · Nền tảng:** `AGENTS.md`, `CLAUDE.md`, `MAP.md`, `context/`, `deliverables/`, `sources/` (chỉ đọc).
- **Tầng 2 · Vận hành:** `SPEC.md`, `PLAN.md`, `CONTRACT_STATUS.md`, `.claude/` (rules, skills, agents, settings).
- **Tầng 3 · Đồng bộ và song song:** git cho HARNESS, `context/git-workflow.md` (branch, worktree, checklist push), `deliverables/codex-inbox/`.

Bắt đầu: đọc `AGENTS.md` → `MAP.md` → 10 dòng cuối `deliverables/session-log.md` → chọn task trong `PLAN.md`.
- Claude Code: `cd` vào thư mục này rồi chạy `claude` (thêm `--add-dir <worktree>` khi làm song song).
- Codex: `codex -C "<thư mục này>" --add-dir "<worktree hoặc REPO>"`, đọc tay `.claude/rules/` và skill cần dùng.

Cuối phiên: Codex chỉ ghi vào `deliverables/codex-inbox/`. **Claude Code chốt phiên** (skill `close-session`) và là bên duy nhất cập nhật harness.
