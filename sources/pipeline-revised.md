# FraudWeave — Revised end-to-end pipeline (v0.1)

> Nguồn: Contract v0.1, mục 2. Đây là bản **thay thế** Mermaid cũ (bản có "TV3 — Core Backend" chứa `/v1/screen`).
> Cập nhật 25/11/2026: nếu brief §5 (QR do TV3 làm) thay đổi gì ở sơ đồ này thì ghi chú bên dưới, không sửa lặng lẽ.

```mermaid
flowchart TB
  subgraph TV2D["TV2 — Discord surfaces"]
    GW["Gateway: message + decoded QR + sender context"]
    SETUP["/setup: Group Trust form"]
    RPT["Context menu: Report to FraudWeave"]
    WARN["Render tier T0-T3 + buttons"]
    MOD["Moderator confirm / dismiss"]
  end

  subgraph API["FastAPI app — shell owned by TV3"]
    SCR["POST /v1/screen (router: TV1)"]
    TR["POST /v1/transfer-risk (TV3)"]
    EVI["POST /v1/evidence (TV3)"]
    GTA["PUT /v1/groups/{guild_hash}/trust (TV3)"]
    CACHE[("Screening cache, in-memory, TTL 30 min: claim_context only")]
  end

  subgraph LIB["fw_common — shared lib (TV3)"]
    NORM["VietQR parser + recipient normaliser + HMAC key"]
  end

  subgraph TV1["TV1 — Detection engine"]
    ENG["engine.screen(): B1-B5 + risk scorer"]
  end

  subgraph DB["Supabase Postgres (TV3)"]
    REG[("Org registry")]
    TRUST[("Group trust")]
    EVD[("Evidence")]
  end

  subgraph SIM["TV2 — Banking simulator / web Check"]
    FORM["Enter account / scan VietQR + optional claimed org"]
    VERD["Verdict screen: Do / Vang / Xam / Xanh"]
  end

  NI["Name inquiry mock (TV3)"]
  POL["Risk policy: decision table (TV3)"]

  GW --> SCR --> ENG
  ENG --> NORM
  ENG --> REG
  ENG --> TRUST
  ENG --> EVD
  ENG --> SCR
  SCR --> CACHE
  SCR --> WARN
  WARN --> MOD --> EVI
  RPT --> EVI
  EVI --> NORM
  EVI --> EVD
  SETUP --> GTA --> TRUST
  WARN -->|"Chuyen khoan button + screening_id"| FORM
  FORM -->|"recipient + screening_id or declared claim"| TR
  TR --> CACHE
  TR --> NORM
  TR --> REG
  TR --> TRUST
  TR --> EVD
  TR --> NI
  TR --> POL --> VERD
```

## Cách đọc

1. **Guardian:** Discord message → `/v1/screen` → `engine.screen()` (đọc registry, trust, evidence) → tier + reasons → claim context cache 30 phút dưới `screening_id` → TV2 hiển thị cảnh báo → mod confirm ghi evidence qua `/v1/evidence`.
2. **SafeTransfer:** người dùng nhập STK / quét QR (từ nút trong cảnh báo hoặc trực tiếp trên web) → `/v1/transfer-risk` → normalise → tra claim context, registry, trust, evidence, name inquiry → decision table → verdict + evidence JSON → TV2 (hoặc trang web tối giản của TV3) hiển thị.
3. **Dùng chung:** mọi ghi vào Evidence đi qua một endpoint ingestion; mọi `recipient_key` đến từ `fw_common`.

## Ghi chú cập nhật sau phân công mới (brief §5)

- VietQR **decode ảnh / parse / encode** do TV3 làm trong `fw_common` và có thêm `POST /v1/qr/decode`; TV2 gửi ảnh (hoặc dùng lại endpoint này) thay vì tự decode ở frontend. Sơ đồ trên chưa vẽ endpoint này.
- TV3 có trang web tối giản "SafeTransfer Check" để demo độc lập; nó đóng vai `FORM` + `VERD` cho đến khi tích hợp với banking simulator của TV2.
- Các mũi tên liên quan Discord (`GW`, `SETUP`, `RPT`, `WARN`, `MOD`, `screening_id`) là PENDING-NEED-CONFIRM-CONTRACT (K06–K10).
