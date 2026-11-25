# Rule: quyền git của agent (D-017)

**TV3 tự commit và push.** Áp dụng cho mọi agent (Claude Code, Codex), ở cả HARNESS, REPO và mọi worktree.

| Được chạy | Không bao giờ chạy |
| --- | --- |
| `git status`, `git diff`, `git log`, `git show <rev>` (xem), `git branch` (liệt kê), `git worktree list`, `git config --get ...` (đọc) | `git add`, `git commit`, `git push`, `git stash`, `git merge`, `git rebase`, `git reset`, `git checkout -b` / `git switch -c` (tạo nhánh), `git tag`, mọi lệnh `gh` |

- Cần tạo nhánh, commit, push, mở PR hay tạo worktree thì **nhắc TV3 tự làm**, kèm lệnh gợi ý và commit message gợi ý (Conventional Commits).
- Cuối phiên: cho TV3 xem `git status` của các repo liên quan và gợi ý commit message. Không làm gì thêm.
- Cấm áp dụng cho mọi shell (Bash, PowerShell) và mọi cách gọi gián tiếp (script, alias, `bash -c`).
- `.claude/settings.json` deny `git add/commit/push` và `gh` cho Bash. Với PowerShell, luật này được thực thi bằng chính quy tắc này [CHƯA KIỂM CHỨNG cú pháp permission riêng cho PowerShell].
