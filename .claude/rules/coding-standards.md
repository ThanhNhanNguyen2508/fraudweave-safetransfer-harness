# Rule: coding standards (REPO)

- **Python 3.11**, môi trường bằng `uv`. FastAPI, **Pydantic v2** là nguồn sự thật cho contract; OpenAPI sinh từ app, không viết tay.
- SQLAlchemy 2 + Alembic cho DB. QR: `opencv-python-headless` để decode, `qrcode` + `pillow` để encode; TLV + CRC tự viết và có test.
- Chất lượng: `ruff` (lint + format) sạch, `pytest` pass, `mypy` ở mức vừa phải.
- Hàm nhỏ, một việc, có **type hints** đầy đủ. Logic policy là pure function, không I/O.
- Enum/giá trị contract viết English `snake_case`; text tiếng Việt cho người dùng chỉ nằm trong `common/reasons.py`.
- Comment, docstring, tên biến/file **tiếng Anh**. Comment giải thích "vì sao", không lặp lại code.
- **Viết test trước hoặc cùng lúc** với code. Mỗi bug sửa xong có test tái hiện.
- Không dead code, không code bị comment-out, không file rác/file rỗng "để sau" (thư mục chưa dùng chỉ có `.gitkeep`).
- Không phụ thuộc mới khi chưa có lý do; thêm dependency thì ghi 1 câu lý do trong PR/session-log.
- Lệnh chuẩn: `uv run pytest`, `uv run ruff check .`, `uv run ruff format --check .`.
