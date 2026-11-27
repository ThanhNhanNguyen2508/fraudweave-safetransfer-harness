# Bộ prompt dựng Harness 3-level cho SafeTransfer (TV3)

Dán **lần lượt** từng prompt vào Claude Code. Chờ prompt trước xong, đọc lại kết quả rồi mới dán prompt sau.
Prompt 1–3 chỉ dựng harness, **chưa viết code sản phẩm**. Prompt 4 mới dựng khung repo.

---

## Bước 0 — Chuẩn bị (làm tay, khoảng 5 phút)

1. Tạo các thư mục nếu chưa có:
   - `C:\Users\Dell\Downloads\MomoTechDay\Implements\Agent Workspace\sources\`
   - `C:\Users\Dell\Downloads\MomoTechDay\Implements\Test-for-agent\`
2. Chép vào `Agent Workspace\sources\`:
   - `00-project-brief.md` (file đi kèm bộ prompt này)
   - `contract-v0.1.md`: mở Claude Doc "FraudWeave — Integration Contract v0.1" → Export → Markdown
   - Ảnh pipeline (`FraudWeave_MVP_Pipeline_v3.png`, ảnh Mermaid) nếu muốn agent tham khảo
3. Mở terminal (PowerShell) và khởi động Claude Code **từ Agent Workspace**, cấp quyền thêm 2 thư mục còn lại:

```powershell
cd "C:\Users\Dell\Downloads\MomoTechDay\Implements\Agent Workspace"
claude --add-dir "C:\Users\Dell\Downloads\MomoTechDay\Implements\Source code-SafeTransfer\FraudWaveAI-Momo" --add-dir "C:\Users\Dell\Downloads\MomoTechDay\Implements\Test-for-agent"
```

> Đường dẫn có dấu cách nên luôn để trong ngoặc kép. Sau Prompt 2, `settings.json` sẽ ghi sẵn 2 thư mục này nên các lần sau chỉ cần gõ `claude`.

---

## Prompt 1 — Tầng 1: Nền tảng (bộ não, bản đồ, bối cảnh)

```text
Tôi muốn bạn làm việc trên thư mục này như một dự án thực thụ, ở mức chuyên nghiệp cao nhất, trong mọi cuộc hội thoại.

BỐI CẢNH: Thư mục hiện tại ("Agent Workspace") là HARNESS (bộ não + quy trình) cho phần SafeTransfer của dự án FraudWeave (MoMo Tech Contest 2026). Tôi là TV3, Backend & Fraud Intelligence Engineer. Đọc KỸ trước khi làm gì:
- sources/00-project-brief.md  (nguồn sự thật: ý tưởng, mục tiêu, phân công, trạng thái contract, kiến trúc 3 thư mục, kế hoạch)
- sources/contract-v0.1.md     (contract chi tiết; §3 của nó đã bị §5 của brief thay thế)

BA THƯ MỤC VÀ VAI TRÒ (tuyệt đối không trộn lẫn):
1. HARNESS = C:\Users\Dell\Downloads\MomoTechDay\Implements\Agent Workspace  → chỉ chứa tài liệu harness, KHÔNG chứa code sản phẩm.
2. REPO = C:\Users\Dell\Downloads\MomoTechDay\Implements\Source code-SafeTransfer\FraudWaveAI-Momo  → git repo sản phẩm, chứa TOÀN BỘ pipeline end-to-end (source, contracts, fixtures, migrations, seed, tests chính thức, scripts, docs). Phải sạch và push được.
3. SANDBOX = C:\Users\Dell\Downloads\MomoTechDay\Implements\Test-for-agent  → nơi agent thử nghiệm, script nháp, output tạm. Không chứa thứ gì pipeline/CI cần. Quy tắc: "xoá SANDBOX mà repo vẫn chạy, CI vẫn pass" thì mới đúng.

Ở bước này CHỈ tạo cấu trúc cơ bản trong HARNESS (không đụng REPO, không viết code):
- CLAUDE.md: tôi là ai, dự án làm gì, cách tôi làm việc, quy tắc bắt buộc. Phải ngắn (≤ 120 dòng); chi tiết thì trỏ sang file khác bằng @import. Dòng đầu tiên import @AGENTS.md.
- AGENTS.md: quy tắc chung cho MỌI agent (Claude Code và Codex đều đọc file này): vai trò 3 thư mục, nguyên tắc "làm được nhiều nhất có thể", quy ước nhãn FROZEN / DRAFT-TV3 / PENDING-NEED-CONFIRM-CONTRACT, quy tắc privacy, quy trình đầu và cuối mỗi phiên.
- MAP.md: bản đồ CẢ 3 thư mục. Mỗi thư mục/file quan trọng chứa gì, tìm thông tin ở đâu, lưu thông tin ở đâu. Cập nhật mỗi khi tạo, xoá hoặc di chuyển file ở bất kỳ thư mục nào.
- context/: mỗi chủ đề một file, dùng cho những gì học được lâu dài (quyết định, lý do, sở thích). Tạo sẵn: decisions.md (log quyết định có ngày, lý do và người quyết), team-and-ownership.md, glossary.md (Claim, C1–C9, verdict, recipient_key, tier, evidence status...).
- deliverables/: mọi thứ bạn tạo cho tôi (báo cáo, ghi chú bàn giao, tóm tắt phiên). Tạo sẵn deliverables/session-log.md.
- sources/: tài liệu thô do tôi cung cấp. Chỉ đọc, KHÔNG sửa.

QUY TẮC:
- Đầu mỗi phiên: đọc CLAUDE.md, AGENTS.md, MAP.md và 10 dòng cuối của deliverables/session-log.md trước khi trả lời.
- Cuối mỗi phiên (hoặc khi tôi nói "chốt phiên"): ghi 3–6 dòng vào session-log.md (làm gì, file nào đổi, việc còn dở, câu hỏi mở) và cập nhật MAP.md.
- Học được điều gì lâu dài thì ghi vào context/. Có quyết định mới thì ghi vào context/decisions.md.
- Không bịa dữ liệu, nguồn hay kết quả. Phân biệt rõ: đã xác minh / giả định / chưa kiểm chứng.
- Trả lời tôi bằng tiếng Việt, giữ nguyên thuật ngữ kỹ thuật tiếng Anh. File harness viết tiếng Việt; tên file, thư mục, code và comment trong code viết tiếng Anh.

Brief đã trả lời phần lớn các câu hỏi về tôi và dự án, nên KHÔNG hỏi lại những gì đã có trong brief. Nếu thật sự còn thiếu gì để viết CLAUDE.md, hỏi TỐI ĐA 3 câu, từng câu một. Xong thì liệt kê các file đã tạo, mỗi file một dòng.
```

---

## Prompt 2 — Tầng 2: Harness hoàn chỉnh (spec, plan, rules, skills, agents, settings)

```text
Hãy phát triển HARNESS thành một hệ thống harness hoàn chỉnh cho SafeTransfer. CHỈ tạo những gì thực sự cần, và mỗi lần thêm gì thì giải thích bằng 1 câu. Vẫn CHƯA viết code sản phẩm trong REPO.

Tạo các file sau trong HARNESS:

1. SPEC.md: chúng ta muốn đạt được gì. Lấy từ brief §1–§3 và §7: phạm vi SafeTransfer, những gì nằm ngoài phạm vi (Guardian engine, Discord bot, UI đẹp của TV2), mục tiêu đo được (≥95% đúng, 0 Xanh sai, p95 ≤150 ms), decision table, 11 golden cases (lấy từ contract §8), định nghĩa "Done" cho từng gate.

2. PLAN.md: các bước thực hiện bám gate G1 29/11, G2 5/12, G3 9/12 (brief §10). Chia thành task nhỏ (≤ ½ ngày), mỗi task có: ID (T-xx), mô tả, file sẽ chạm trong REPO, test chứng minh xong, phụ thuộc, trạng thái [todo/doing/done/blocked-pending]. Task phụ thuộc vào contract PENDING thì vẫn đưa vào, nhưng ghi rõ "làm với stub trước", kèm ID contract (K06–K12).

3. CONTRACT_STATUS.md: bảng K01–K12 lấy từ brief §6, thêm cột "Stub/adapter trong code" và "Ngày cập nhật". Đây là nơi DUY NHẤT theo dõi trạng thái contract.

4. CLAUDE.local.md: sở thích cá nhân (trả lời tiếng Việt, văn phong tự nhiên, ngắn gọn; luôn nói rõ mức độ chắc chắn). Thêm vào .gitignore của HARNESS.

5. .claude/rules/ (và import từng file từ CLAUDE.md bằng @, để chắc chắn được nạp):
   - folder-placement.md: file nào đặt ở HARNESS / REPO / SANDBOX, kèm quy tắc phân xử "xoá SANDBOX mà repo vẫn chạy". Test chính thức (unit/contract/golden) nằm trong REPO/tests; test nháp, benchmark thử, notebook khám phá nằm trong SANDBOX.
   - contract-pending.md: cách làm việc với phần chưa chốt. Mọi điểm phụ thuộc contract PENDING phải đi qua interface/adapter + stub. Trong code, đánh dấu đúng định dạng `# PENDING-NEED-CONFIRM-CONTRACT(Kxx): <mô tả>`. Không được tự ý "chốt" thay người khác. Khi đổi một contract DRAFT-TV3 phải cập nhật CONTRACT_STATUS.md, context/decisions.md và fixtures.
   - privacy-security.md: không lưu message text, STK thô, Discord ID, số tiền chính xác; chỉ lưu recipient_key (HMAC), số đã che, amount_band. Secrets chỉ nằm trong REPO/.env. Không bao giờ in secret ra log hay chat. Dữ liệu demo phải là synthetic và có cờ.
   - coding-standards.md: Python 3.11, FastAPI, Pydantic v2 làm nguồn sự thật cho contract, ruff + pytest, hàm nhỏ có type hints, không dead code, không file rác, comment tiếng Anh, viết test trước hoặc cùng lúc với code.
   - definition-of-done.md: một task chỉ "done" khi test trong REPO pass, ruff sạch, MAP.md và PLAN.md đã cập nhật, session-log đã ghi.

6. .claude/skills/ (mỗi skill một thư mục có SKILL.md, mô tả rõ khi nào dùng):
   - implement-task: quy trình làm 1 task trong PLAN.md: đọc task → kiểm tra CONTRACT_STATUS → viết/cập nhật test trong REPO/tests → implement → chạy test → cập nhật PLAN/MAP/session-log → tóm tắt 5 dòng.
   - contract-audit: grep mọi marker PENDING-NEED-CONFIRM-CONTRACT trong REPO, đối chiếu với CONTRACT_STATUS.md, báo phần lệch.
   - add-golden-case: thêm 1 golden case (fixture JSON request + expected) vào REPO/contracts/fixtures và test tương ứng.
   - handoff-note: viết ghi chú bàn giao cho TV1 hoặc TV2 (endpoint, ví dụ request/response, fixtures, việc họ cần chốt) vào deliverables/handoff/.
   - sandbox-experiment: chạy thử nghiệm trong SANDBOX (mỗi thử nghiệm một thư mục có ngày + README 3 dòng), không bao giờ ghi vào REPO.

7. .claude/agents/:
   - code-reviewer: review "mắt mới" một thay đổi trong REPO, đối chiếu SPEC, contract, privacy rules, coding standards; chỉ đọc, không sửa.
   - contract-auditor: chỉ đọc, dùng để kiểm tra contract trước mỗi gate.

8. .claude/settings.json: permissions.additionalDirectories gồm REPO và SANDBOX (đường dẫn Windows đầy đủ); deny đọc/ghi mọi file .env (Read(**/.env), Edit(**/.env)); cho phép các lệnh an toàn hay dùng (uv, pytest, ruff, git status/diff/log). Không cho phép git push hay xoá hàng loạt khi chưa hỏi tôi.

9. .claude/workflows/ chỉ tạo nếu Claude Code bản này hỗ trợ; nếu không, ghi quy trình vào skill và nói rõ lý do bỏ qua. Bỏ qua .mcp.json và output-styles/ trừ khi bạn thấy lý do cụ thể (giải thích 1 câu).

10. README.md của HARNESS: 15 dòng cho người mới (hoặc Codex) biết bắt đầu từ đâu.

Sau đó cập nhật MAP.md (mỗi file một dòng mô tả). Cuối cùng chạy kiểm tra tự thân: liệt kê cây thư mục HARNESS và xác nhận không có file nào thừa, không có code sản phẩm trong HARNESS.
```

---

## Prompt 3 — Tầng 3: Đồng bộ và chạy song song

```text
Tôi muốn dự án đồng bộ được ở mọi nơi, và nhiều phiên agent làm việc cùng lúc mà không phá code.

1. HARNESS: khởi tạo Git và tạo một PRIVATE repo trên GitHub của tôi tên "fraudweave-safetransfer-harness" (hướng dẫn từng bước nếu cần, tôi tự đăng nhập). Trước lần push đầu tiên, kiểm tra .gitignore đã loại trừ CLAUDE.local.md, .claude/settings.local.json, mọi .env, thư mục tạm. Cho tôi xem danh sách file sẽ được push và CHỜ tôi đồng ý.

2. REPO (đã clone, có remote sẵn, hiện chỉ có README): KHÔNG push gì ở bước này. Chỉ đề xuất và ghi vào HARNESS/context/git-workflow.md:
   - Chiến lược branch: main luôn chạy được; nhánh của tôi đặt tên tv3/<module>-<mô tả ngắn>; gộp qua Pull Request; commit message theo Conventional Commits.
   - .gitignore chuẩn cho Python/uv/FastAPI, có .env, __pycache__, .venv, .pytest_cache, *.log, dữ liệu tạm.
   - Checklist trước mỗi push: test pass, ruff sạch, không có secret (grep các pattern key), không có file nào trỏ tới SANDBOX.

3. Chạy song song: đề xuất quy tắc dùng git worktree cho REPO. Mỗi phiên agent = 1 worktree = 1 nhánh = 1 vùng module (ví dụ: fw_common, evidence, safetransfer-api, web-check). Đặt worktree ở C:\Users\Dell\Downloads\MomoTechDay\Implements\worktrees\<tên-nhánh> (ngoài REPO, ngoài SANDBOX). Nói rõ file .env được chép vào worktree thế nào (dùng .worktreeinclude nếu Claude Code bản này hỗ trợ, nếu không thì dùng script trong REPO/scripts). Hai phiên không được cùng sửa fw_common/contracts.py; ai đổi contract thì đổi trong một nhánh riêng.

4. Giải thích cho tôi trong 3 câu đơn giản: cách lưu công việc và lấy nó về trên máy khác (cho cả HARNESS và REPO).

5. Cập nhật MAP.md, AGENTS.md (phần git và worktree), session-log.md.
```

---

## Prompt 4 — Dựng khung REPO (bắt đầu implement)

> Chỉ chạy sau khi Prompt 1–3 xong và bạn đã đọc lại SPEC.md, PLAN.md.

```text
Bây giờ dựng KHUNG cho REPO theo SPEC.md và PLAN.md. Dùng skill implement-task cho phần có logic. Mục tiêu của bước này: repo chạy được `uv run pytest` và `uv run uvicorn` với endpoint health, và có đủ chỗ cho toàn bộ pipeline end-to-end của SafeTransfer.

Cấu trúc đề xuất (được phép chỉnh nếu có lý do, nhưng phải giải thích 1 câu):

FraudWaveAI-Momo/
├── README.md               # mục tiêu, cách chạy, kiến trúc 1 sơ đồ Mermaid, trạng thái contract (link)
├── AGENTS.md               # ≤ 40 dòng: cấu trúc, lệnh chạy, quy tắc, cho đồng đội và agent (không lặp lại harness)
├── pyproject.toml          # uv, ruff, pytest config
├── .env.example            # FW_HMAC_KEY, DATABASE_URL, API_KEYS, APP_MODE
├── .gitignore
├── contracts/
│   ├── CONTRACT_STATUS.md  # bản sao gọn của bảng trong HARNESS (repo phải tự đứng được)
│   ├── openapi.json        # sinh từ app, không viết tay
│   └── fixtures/{normalize,qr,transfer}/
├── src/fraudweave/
│   ├── common/             # = fw_common: contracts.py, normalize.py, banks.py, hashing.py, reasons.py
│   │   └── vietqr/         # tlv.py, crc.py, parse.py, encode.py, decode_image.py
│   ├── api/                # main.py, deps.py, errors.py, routers/{health,qr,transfer,evidence,registry,groups}.py
│   ├── safetransfer/       # service.py, policy.py, claim_providers.py, name_inquiry_mock.py
│   ├── evidence/           # repository.py, ingestion.py
│   ├── registry/           # repository.py
│   ├── storage/            # db.py, models.py
│   └── integrations/       # guardian_adapter.py (PENDING K06/K07, chỉ có interface + stub)
├── migrations/             # alembic
├── data/seed/              # banks.json, registry_demo.json, groups_demo.json, evidence_demo.json (synthetic, có cờ)
├── web/safetransfer-check/ # index.html tĩnh: nhập STK/tải ảnh QR + chọn "tự xưng là" → hiển thị verdict
├── scripts/                # seed_db.py, gen_demo_qr.py, export_openapi.py
├── tests/{unit,contract,golden}/
└── docs/                   # architecture.md (pipeline Mermaid), safetransfer-policy.md

Yêu cầu:
- Tạo khung + pyproject + .gitignore + .env.example + README + AGENTS.md + app FastAPI chạy được với /v1/health + 1 test pass.
- Tạo SẴN các file Pydantic trong common/contracts.py theo contract §4 (DRAFT-TV3). Phần PENDING thì để interface + stub có marker PENDING-NEED-CONFIRM-CONTRACT(Kxx).
- CHƯA implement logic normalise/QR/policy. Đó là T-xx tiếp theo trong PLAN.md.
- Không tạo file rỗng "để sau". Thư mục chưa dùng thì chỉ cần .gitkeep và 1 dòng trong MAP.md.
- Không commit, không push. Cho tôi xem `git status` và cây thư mục rồi chờ tôi duyệt.
- Cập nhật MAP.md, PLAN.md, session-log.md trong HARNESS.
```

---

## Prompt mẫu cho mỗi phiên làm việc sau đó

```text
Phiên mới. Đọc CLAUDE.md, AGENTS.md, MAP.md, PLAN.md, CONTRACT_STATUS.md và cuối session-log.md.
Làm task T-__ bằng skill implement-task. Nếu task chạm vào contract PENDING thì dùng stub, đánh marker, và KHÔNG tự chốt.
Thử nghiệm nháp thì đặt trong SANDBOX. Xong thì cho tôi xem diff + kết quả test, chưa commit.
```

```text
Chốt phiên: cập nhật session-log, PLAN, MAP; chạy skill contract-audit; liệt kê câu hỏi cần hỏi TV1/TV2.
```

Phiên mới. Đọc CLAUDE.md, AGENTS.md, MAP.md, PLAN.md, CONTRACT_STATUS.md và 10 dòng cuối session-log.md.
Tôi đã commit T-15 và T16 . Làm task T-18 và T-19 bằng skill implement-task.
Nếu task cần Opus (T-07a, T-17, nhãn eval T-24) mà phiên đang Sonnet thì dừng và nhắc tôi /model.
Task chạm contract PENDING thì dùng stub + marker, KHÔNG tự chốt. Nháp thì để trong SANDBOX.
Cuối task: gọi code-reviewer review diff, rồi cho tôi xem diff + kết quả test. Chưa commit.


Lệnh khi bật claude code : 
cd "C:\Users\Dell\Downloads\MomoTechDay\Implements\Agent Workspace"

claude --resume b4857841-9b0a-4ea2-b234-1fa53109554a --add-dir "C:\Users\Dell\Downloads\MomoTechDay\Implements\Source code-SafeTransfer\FraudWaveAI-Momo"