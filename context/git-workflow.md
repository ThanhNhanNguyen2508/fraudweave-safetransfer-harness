# Git workflow, worktree và đồng bộ

> Trạng thái: phần quy ước của TV3 **đã chốt** (D-013); quy ước chung chờ nhóm xác nhận. Worktree **duyệt có điều kiện**, phải kiểm chứng thật ở T-04 (D-014). Commit dùng email noreply (D-016). Mọi thứ cho REPO ở đây chỉ là đề xuất; file thật sẽ tạo trong REPO ở T-04. Cập nhật 2026-11-25.

## 1. HARNESS (repo riêng, private)

- Remote dự kiến: GitHub private `fraudweave-safetransfer-harness` của TV3. Nhánh duy nhất `main`; không cần PR.
- **TV3 tự commit và push** cả HARNESS và REPO (D-017). Agent chỉ chạy `git status/diff/log` và gợi ý commit message ở bước "chốt phiên" (skill `close-session`). Agent cũng không tạo nhánh hay worktree; script `new_worktree.ps1` do TV3 chạy.
- Commit message: `docs(harness): <tóm tắt phiên>` (Conventional Commits).
- Không bao giờ commit: `CLAUDE.local.md`, `.claude/settings.local.json`, `.env*`, thư mục tạm (đã có trong `.gitignore`).

## 2. REPO: chiến lược branch

- `main` **luôn chạy được**: `uv run pytest` pass, `ruff` sạch. Không commit thẳng lên `main`.
- Nhánh của TV3: `tv3/<module>-<mô-tả-ngắn>`, chữ thường, kebab-case. Ví dụ: `tv3/vietqr-parse-tag38`, `tv3/safetransfer-policy-table`.
- `<module>` lấy từ danh sách: `fw-common`, `vietqr`, `contracts`, `storage`, `evidence`, `registry`, `groups`, `safetransfer`, `api`, `web-check`, `eval`, `scripts`, `docs`, `ci`.
- Mỗi nhánh tương ứng 1 task T-xx (hoặc một nhóm nhỏ liền nhau). Gộp vào `main` qua **Pull Request**, ưu tiên squash merge. PR ghi T-xx, Kxx liên quan và output test.
- Nên bật branch protection cho `main` (cần quyền owner của remote).
- **Commit message: Conventional Commits.** `<type>(<scope>): <mô tả tiếng Anh, thì hiện tại>`.
  - type: `feat`, `fix`, `test`, `refactor`, `docs`, `chore`, `ci`, `perf`. Scope = `<module>`.
  - Ví dụ: `feat(vietqr): parse tag 38 into bank BIN and account`, `test(safetransfer): add golden R2`.
  - Breaking contract: thêm `!` (`feat(contracts)!: rename verdict field`) và làm theo `.claude/rules/contract-pending.md`.

## 3. REPO: `.gitignore` đề xuất (tạo ở T-04)

```gitignore
# Secrets
.env
.env.*
!.env.example

# Python / uv
__pycache__/
*.py[cod]
.venv/
.python-version.local
*.egg-info/
build/
dist/

# Tooling caches
.pytest_cache/
.ruff_cache/
.mypy_cache/
.coverage
htmlcov/

# Logs and temporary data
*.log
tmp/
.tmp/
data/tmp/
data/local/
*.sqlite3
out/

# Claude Code / editors / OS
.claude/settings.local.json
.claude/worktrees/
.vscode/
.idea/
.DS_Store
Thumbs.db
```

`uv.lock` **được commit** để môi trường giống nhau ở mọi máy và mọi worktree.

## 4. REPO: checklist trước mỗi push

Đề xuất gói thành `scripts/check_prepush.ps1` ở T-04, vì hook bảo vệ secret của Claude Code chặn mọi lệnh shell nhắc tới tên file `.env`. Từng bước:

1. `uv run pytest`: pass toàn bộ.
2. `uv run ruff check .` và `uv run ruff format --check .`: sạch.
3. Không có file secret nào được track: `git ls-files` không chứa `.env` (trừ `.env.example`), `*.pem`, `*.key`.
4. Grep pattern secret trong file đã track, kết quả phải rỗng:
   ```
   git grep -n -I -E "(FW_HMAC_KEY|API_KEYS|DATABASE_URL)\s*=\s*['\"]?[^'\"\s<]{8,}" -- . ":!*.example"
   git grep -n -I -E "postgres(ql)?://[^:/\s]+:[^@\s]+@"
   git grep -n -I -E "eyJ[A-Za-z0-9_-]{20,}\.[A-Za-z0-9_-]{20,}\.[A-Za-z0-9_-]{10,}"   # JWT / Supabase keys
   git grep -n -I -E "-----BEGIN [A-Z ]*PRIVATE KEY-----"
   ```
5. Không trỏ tới SANDBOX hoặc HARNESS: `git grep -n -i -E "Test-for-agent|Agent Workspace"` phải rỗng.
6. Không có STK hoặc dữ liệu thật trong fixtures/seed (mọi bản ghi có cờ synthetic).
7. Nếu đổi contract: `CONTRACT_STATUS.md`, `context/decisions.md`, fixtures và `openapi.json` đã đồng bộ.
8. `git status` sạch, nhánh đúng tên, TV3 đã đồng ý push.

## 5. Chạy song song bằng git worktree

### Quy tắc
- **1 phiên agent = 1 worktree = 1 nhánh = 1 vùng module.** Ví dụ chia vùng: `fw-common`, `evidence`, `safetransfer-api`, `web-check`.
- Vị trí: `C:\Users\Dell\Downloads\MomoTechDay\Implements\worktrees\<tên-nhánh-đổi-/-thành-->`
  ví dụ nhánh `tv3/vietqr-parse-tag38` → `worktrees\tv3-vietqr-parse-tag38`. Nằm ngoài REPO, ngoài SANDBOX. Đổi `/` thành `-` để thư mục phẳng, dễ quét.
- Thư mục REPO gốc giữ nhánh `main`, chỉ dùng để pull, review và tạo worktree. Không code trực tiếp ở đó khi có phiên song song.
- Mở hoặc đóng worktree → cập nhật bảng **"Worktree đang mở"** trong `MAP.md` (nhánh, đường dẫn, vùng module, agent, giữ khoá contract hay không).
- **Khoá contract:** chỉ **một** nhánh `tv3/contracts-<mô-tả>` tại một thời điểm được sửa `src/fraudweave/common/contracts.py`, `contracts/openapi.json` và schema fixtures. Phiên khác cần đổi contract thì dừng, báo TV3, chờ nhánh contract merge rồi `git rebase origin/main`.
- Hai worktree không cùng sửa một file. Nếu bắt buộc thì làm tuần tự: merge nhánh A trước, rebase nhánh B sau.
- Xong việc: PR merge → `git worktree remove <path>` → `git branch -d <branch>` → xoá dòng trong MAP.md.

### File `.env` vào worktree
- `.env` không được git track nên worktree mới **không có** nó.
- Claude Code có `.worktreeinclude` (liệt kê file gitignored cần chép sang worktree), nhưng chỉ áp dụng khi chạy `claude --worktree`. Lệnh đó tạo worktree **bên trong** REPO (`.claude/worktrees/`), trái với vị trí đã chọn [GIẢ ĐỊNH theo tài liệu Claude Code, chưa thử trên máy này]. Codex cũng không dùng file này.
- Vì vậy **không dùng `claude --worktree` cho REPO**. Dùng script `REPO/scripts/new_worktree.ps1` (tạo ở T-04). Script đọc danh sách file cần chép từ `REPO/.worktreeinclude` (một dòng một đường dẫn, mặc định chỉ có `.env`), để sau này nếu muốn dùng `--worktree` thì vẫn chung một danh sách.
- Chép (copy), không symlink, vì symlink trên Windows cần Developer Mode. Script không bao giờ in nội dung file.

Phác thảo script (sẽ viết thật và test ở T-04):

```powershell
# scripts/new_worktree.ps1 — create a worktree outside the repo and copy local-only files.
param(
    [Parameter(Mandatory = $true)][string]$Branch,   # e.g. tv3/vietqr-parse-tag38
    [string]$Base = "main"
)
$ErrorActionPreference = "Stop"
$mainRepo = Split-Path (git rev-parse --path-format=absolute --git-common-dir) -Parent
$implements = Split-Path (Split-Path $mainRepo -Parent) -Parent
$target = Join-Path $implements ("worktrees\" + ($Branch -replace "/", "-"))
if (Test-Path $target) { throw "Worktree path already exists: $target" }

git -C $mainRepo fetch origin
git -C $mainRepo worktree add -b $Branch $target "origin/$Base"

$includeFile = Join-Path $mainRepo ".worktreeinclude"
if (Test-Path $includeFile) {
    Get-Content $includeFile | Where-Object { $_ -and -not $_.StartsWith("#") } | ForEach-Object {
        $src = Join-Path $mainRepo $_
        if (Test-Path $src) { Copy-Item $src (Join-Path $target $_) }   # never print contents
    }
}
Write-Output "Worktree ready: $target"
```

Việc phải xử lý thêm ở T-04: nhánh đã tồn tại trên remote (máy khác tạo trước) thì dùng `worktree add $target $Branch`, không dùng `-b`.

### Khởi động agent trong worktree (luôn đứng ở HARNESS để nạp bộ não)
```powershell
cd "C:\Users\Dell\Downloads\MomoTechDay\Implements\Agent Workspace"
# Claude Code
claude --add-dir "C:\Users\Dell\Downloads\MomoTechDay\Implements\worktrees\tv3-vietqr-parse-tag38"
# Codex
codex -C "C:\Users\Dell\Downloads\MomoTechDay\Implements\Agent Workspace" --add-dir "C:\Users\Dell\Downloads\MomoTechDay\Implements\worktrees\tv3-vietqr-parse-tag38"
```
Câu đầu tiên của phiên phải nói rõ: "Phiên này làm T-xx trong worktree `<path>`, nhánh `<branch>`". Agent chỉ sửa code trong worktree đó, không sửa REPO gốc.

## 6. Đồng bộ giữa các máy

- **HARNESS:** clone `fraudweave-safetransfer-harness` vào `...\Implements\Agent Workspace`. Tạo lại `CLAUDE.local.md` bằng tay. Nếu đường dẫn máy mới khác, sửa `additionalDirectories` trong `.claude/settings.json` (đang dùng đường dẫn tuyệt đối).
- **REPO:** clone `FraudWaveAI-Momo` vào `...\Implements\Source code-SafeTransfer\`, chạy `uv sync`, tạo `.env` từ `.env.example` (secret lấy từ password manager hoặc kênh riêng, không qua git, không qua chat).
- **Công việc dở trên nhánh:** push nhánh `tv3/...` lên remote (sau khi TV3 đồng ý). Máy khác chạy `git fetch` rồi tạo worktree từ nhánh đó.
- **SANDBOX** không đồng bộ, vì nó chỉ là nháp.
