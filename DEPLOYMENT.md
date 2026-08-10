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
| Public URL | Chưa có public cloud URL; đang dùng local fallback tại `http://localhost:8000` |
| Platform | Local fallback bằng Docker Compose; cấu hình cloud đã chuẩn bị cho Railway / Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set

Ghi tên biến và nguồn giá trị, không ghi giá trị secret:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | Có | Docker Compose/platform gán, mặc định 8000 khi chạy local |
| `AGENT_API_KEY` | Có | đặt trong `.env` local hoặc dashboard cloud, không nằm trong repo |
| `REDIS_URL` | Có | local fallback dùng `redis://redis:6379/0` trong Docker Compose |
| `RATE_LIMIT_PER_MINUTE` | Có | 10 |
| `MONTHLY_BUDGET_USD` | Có | 10.0 |
| `LOG_LEVEL` | Có | INFO |
| `LOCAL_FALLBACK` | Có | `true` trong `.env` local để CP5 kiểm tra localhost |

## Lệnh Kiểm Tra Đã Chạy

```powershell
docker compose up -d --build
docker compose ps
curl.exe -i http://localhost:8000/health
curl.exe -i http://localhost:8000/ready
.\.venv\Scripts\python.exe -c "import httpx; r=httpx.post('http://localhost:8000/ask', json={'question':'Hello'}, timeout=20); print(r.status_code, r.text)"
```

## Kết Quả Chạy Thật

```text
docker compose ps:
agent: Up, port 0.0.0.0:8000->8000/tcp
redis: Up, healthy, port 0.0.0.0:6379->6379/tcp

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

Ảnh local fallback đã đặt tại:

- `screenshots/local-fallback.png`

## Nếu Dùng Phương Án Dự Phòng

Lý do dùng phương án dự phòng: máy hiện chưa có Railway CLI trong PATH và không có session đăng nhập Railway/Render để tạo public HTTPS URL thật từ dòng lệnh. Stack local bằng Docker Compose đã chạy được, `/health` và `/ready` đều trả 200, `/ask` không có API key trả 401.
