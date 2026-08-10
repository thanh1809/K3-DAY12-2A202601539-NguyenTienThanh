# Thông Tin Deploy — Checkpoint 5

> Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Tiến Thành |
| Mã học viên | 2A202601539 |
| Repo | https://github.com/thanh1809/K3-DAY12-2A202601539-NguyenTienThanh |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-u763.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set

Ghi tên biến và nguồn giá trị, không ghi giá trị secret:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | Có | Render tự gán |
| `AGENT_API_KEY` | Có | đặt trong `.env` local hoặc dashboard cloud, không nằm trong repo |
| `REDIS_URL` | Có | Render Key Value `day12-redis`, lấy từ `render.yaml` |
| `RATE_LIMIT_PER_MINUTE` | Có | 10 |
| `MONTHLY_BUDGET_USD` | Có | 10.0 |
| `LOG_LEVEL` | Có | INFO |
| `LOCAL_FALLBACK` | Có | `false` trong `.env` local để CP5 kiểm tra public URL |

## Lệnh Kiểm Tra Đã Chạy

```powershell
curl.exe -i https://day12-agent-u763.onrender.com/health
curl.exe -i https://day12-agent-u763.onrender.com/ready
.\.venv\Scripts\python.exe -c "import httpx; r=httpx.post('https://day12-agent-u763.onrender.com/ask', json={'question':'Hello'}, timeout=60); print(r.status_code, r.text)"
```

## Kết Quả Chạy Thật

```text
GET /health:
HTTP/1.1 200 OK
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready:
HTTP/1.1 200 OK
{"status":"ready","redis":true}

POST /ask without API key:
401 {"detail":"invalid or missing API key"}
```

## Ảnh Chụp Màn Hình

Ảnh kiểm tra deploy đã đặt tại:

- `screenshots/local-fallback.png`

## Nếu Dùng Phương Án Dự Phòng

Không dùng phương án dự phòng. Service đã deploy lên Render tại `https://day12-agent-u763.onrender.com`; `/health` và `/ready` đều trả 200, `/ask` không có API key trả 401.
