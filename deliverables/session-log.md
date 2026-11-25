# Session log

> Mỗi phiên ghi 3–6 dòng ở **cuối file**: làm gì · file nào đổi · việc còn dở · câu hỏi mở.
> Agent đầu phiên đọc 10 dòng cuối file này.

## 2026-11-25 · Phiên 1 · Harness tầng 1 + tầng 2
- Làm: đọc brief, contract v0.1, pipeline-revised; dựng harness tầng 1 (CLAUDE, AGENTS, MAP, context/, deliverables/) và tầng 2 (SPEC, PLAN, CONTRACT_STATUS, rules, skills, agents, settings, README).
- File đổi: toàn bộ file mới trong HARNESS (xem MAP.md). Không đụng REPO, SANDBOX.
- Còn dở: T-03 (harness tầng 3: git + worktree), T-04 (scaffold REPO).
- Câu hỏi mở: TV3 duyệt D-007 (cách gọi golden vs gate) và D-008 (bỏ workflows/); chưa rõ ai sở hữu remote GitHub `ThanhNhanNguyen2508/FraudWaveAI-Momo`; các câu hỏi freeze ở contract §10.

## 2026-11-25 · Phiên 2 · Harness tầng 3 (git, worktree, Codex ↔ Claude Code)
- Làm: `git init -b main` cho HARNESS (chưa commit, chưa remote); viết `context/git-workflow.md`; thêm `codex-inbox/` + skill `close-session` (chỉ Claude Code chốt phiên); cập nhật AGENTS §7/§7b/§9, CLAUDE, README, settings (thêm `worktrees`), MAP §4, PLAN T-03/T-04, decisions D-009…D-012.
- Kiểm chứng: `gh` chưa cài; `codex -C/--add-dir` và `claude --add-dir` có thật (đã xem `--help`); hook chặn lệnh có chữ `.env`, nên kiểm `.env` bằng cách đọc `.gitignore`.
- Còn dở: T-03 chờ TV3 tạo private repo và đồng ý push; T-04 tạo `.gitignore`, `.worktreeinclude`, `scripts/new_worktree.ps1`, `check_prepush.ps1` trong REPO.
- Câu hỏi mở: TV3 duyệt D-010/D-011; remote REPO có phải tài khoản của TV3 không; branch protection `main` cần owner bật.
- Bổ sung: TV3 duyệt 32 file, chọn Cách A (gh), xác nhận remote REPO là tài khoản của mình. Ghi D-013 (chốt D-010 phần TV3), D-014 (D-011 có điều kiện T-04), D-015 (không sửa hook: đã thử trong SANDBOX, `.env.example` ghi/sửa/đọc được, `.env` bị chặn; thư mục thử đã xoá), D-016 (email noreply). Chờ TV3 cài và đăng nhập gh, xác nhận email noreply, rồi mới commit và push.

## 2026-11-25 · Phiên 2 (tiếp) · Áp dụng review PLAN/SPEC
- Làm: viết lại PLAN.md (T-03 manual; T-04a/T-04b; T-06b/c/d; T-07a; T-11/T-12 ra khỏi G1; T-19 deps mới; T-18 khớp alias chính xác; T-23 ô chọn demo group; T-24 quy tắc eval); SPEC §2/§3/§4/§4.1/§6/§7; CONTRACT_STATUS (câu hỏi Xanh vs C7–C9); decisions D-018…D-026; MAP.
- Kiểm chứng: Docker CLI 29.9.3 + Compose v5.7.0 đã cài, nhưng engine **chưa chạy** (không kết nối được `dockerDesktopLinuxEngine`).
- Còn dở: T-03 (TV3 cài gh, đăng nhập, xác nhận noreply → push); bắt đầu Prompt 4 / T-04a khi TV3 cho phép.
- Câu hỏi mở: duyệt ngưỡng ≥ 30% ca nhiễu (D-023); bước gộp khoảng trắng khi chuẩn hoá alias (SPEC §4.1); D-017 không dùng (review yêu cầu đánh số từ D-018).

## 2026-11-25 · Phiên 3 · Luật git D-017 + T-04a khung REPO
- Làm: D-017 (TV3 tự commit/push; agent chỉ git status/diff/log), D-027, D-028; rule `git-permissions.md`; settings deny git add/commit/push + gh; sửa AGENTS/CLAUDE/close-session/git-workflow/SPEC/PLAN. REPO (nhánh `tv3/scaffold-repo`): khung T-04a (pyproject, .gitignore, .env.example, docker-compose, README, AGENTS, app + `/v1/health`, 2 test, `.gitkeep`). Không làm T-04b/T-05.
- Kiểm chứng: venv Python 3.11 tạm trong scratchpad: `pytest -W error` 2 pass, ruff check + format sạch, uvicorn `/v1/health` 200; `docker compose config` hợp lệ. Dev dep đổi `httpx` → `httpx2` (Starlette 1.10 deprecate httpx; đã kiểm metadata: Pydantic Services bảo trì).
- Chưa làm được: `uv` chưa cài (chưa có `uv.lock`, chưa chạy `uv run`); Docker engine không chạy (không có tiến trình Docker Desktop) → chưa test Postgres. T-04a = doing.
- Câu hỏi mở: email local cả hai repo đang là nguyên chuỗi mẫu `<ID>+ThanhNhanNguyen2508@...` (chưa thay ID); PowerShell không có luật deny riêng cho git (chỉ có rule văn bản).

## 2026-11-25 · Phiên 4 · D-029 + hoàn tất T-04a
- Làm: D-029 (bước Docker tách khỏi Done của T-04a, chuyển sang T-14); `uv sync` sinh `REPO/uv.lock`; PLAN (T-04a done, T-14 thêm `docker compose up -d db` + `pg_isready`), MAP.
- Kiểm chứng: uv 0.13.10; `uv run pytest -W error` 2 pass; `ruff check` + `ruff format --check` sạch; `uv run uvicorn ... --factory` → `/v1/health` 200; `git check-ignore uv.lock` không khớp (không bị ignore).
- Ghi chú: uv cài qua winget nhưng shell của agent chưa nạp PATH mới → gọi bằng đường dẫn đầy đủ; mở terminal mới là có `uv`.
- Còn dở: TV3 commit T-04a trên `tv3/scaffold-repo`; tiếp theo T-05. Docker chờ tới T-14.
