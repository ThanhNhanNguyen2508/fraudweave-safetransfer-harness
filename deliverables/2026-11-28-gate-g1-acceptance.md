# Nghiệm thu Gate G1 — 2026-11-28

> Hạn G1: 29/11. Căn cứ: `SPEC.md` §6 (không đổi). Nhánh REPO: `tv3/api-health-ratelimit` (gồm T-25, chưa commit tại thời điểm viết; nền là `a40173e` T-18/T-19).
> Người chạy test DB: **TV3**, Postgres local (docker-compose), `FW_TEST_DATABASE_URL` → `fraudweave_test`.

## Tiêu chí SPEC §6 (nguyên văn)

> `fw_common` có test: normalise + **VietQR parse + CRC**; DB schema + migrations + seed synthetic chạy được; `POST /v1/transfer-risk` pass **golden G1, R1, Y1, E1 trên DB thật = Postgres local (docker-compose)**; `ruff` sạch; `/v1/health` trả `policy_version`

## Đối chiếu

| Tiêu chí | Task | Bằng chứng | Kết quả |
| --- | --- | --- | --- |
| normalise có test | T-07, T-08 | `tests/unit/test_{banks,normalize,hashing,text}.py` nằm trong regression | ✅ |
| VietQR parse + CRC có test | T-09, T-10 | `tests/unit/test_{crc,tlv,vietqr_parse}.py`, `tests/contract/test_qr_fixtures.py` | ✅ |
| DB schema + migrations + seed synthetic | T-14, T-15 | `tests/unit/test_migrations.py`, `test_seed.py` chạy trên Postgres (không skip) | ✅ |
| `transfer-risk` pass golden G1, R1, Y1, E1 trên Postgres local | T-19 | `tests/golden/test_transfer_golden.py`: chạy đủ 12 golden (gồm G1, R1, Y1, E1) | ✅ |
| `ruff` sạch | — | `ruff check` + `ruff format --check` | ✅ |
| `/v1/health` trả `policy_version` | T-25 | `tests/unit/test_health.py`: trả `st-policy-0.2` + `contract_version` `0.1` (D-051) | ✅ |

## Kết quả test

| Lần chạy | Ai | Môi trường | Kết quả |
| --- | --- | --- | --- |
| `pytest tests/golden` | TV3 | Postgres local | **26 passed** in 20.60s [ĐÃ XÁC MINH, TV3 báo] |
| `pytest -W error` toàn bộ | TV3 | Postgres local | **866 passed**, 0 failed, 0 error, 0 skipped, 166.34s [ĐÃ XÁC MINH, TV3 báo] |
| `pytest` toàn bộ | agent | không có DB | 707 passed, 159 skipped (đều là test cần DB) [ĐÃ XÁC MINH]; 707 + 159 = 866, khớp số của TV3 |
| `ruff check .`, `ruff format --check .`, `export_openapi.py --check` | agent | — | sạch, OpenAPI không drift [ĐÃ XÁC MINH] |
| `contract-audit` | agent | — | 4 marker hợp lệ (K07 ×3, K06 ×1) khớp PENDING; bản sao `contracts/CONTRACT_STATUS.md` khớp; 12 fixture có đủ request + expected; 0 lệch [ĐÃ XÁC MINH] |

## Kết luận

**Gate G1 đạt** theo SPEC §6. Điều kiện: TV3 commit T-25, vì phần health `policy_version` nằm trong thay đổi này.

## Ngoài phạm vi G1, ghi lại để không quên

- `main` của REPO vẫn chỉ có "Initial commit"; code G1 nằm trên các nhánh `tv3/*`. Merge qua PR là việc của TV3.
- T-06d (họp chốt K06–K10) hạn **trước 30/11**, đang `manual`.
- Mốc giữa 1/12: T-11, T-12 (D-019), T-13, T-20 (nghiệm thu), T-21, T-22, T-23, T-24.
- Rate limit đếm trong bộ nhớ một process: khi deploy (K12) phải chạy 1 worker (D-051).
