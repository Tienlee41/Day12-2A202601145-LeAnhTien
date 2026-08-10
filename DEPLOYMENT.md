# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Lê Anh Tiến |
| Mã học viên | 2A202601145 |
| Repo | https://github.com/Tienlee41/Day12-2A202601145-LeAnhTien |

## Service

| Mục | Nội dung |
|-----|----------|
| Base URL kiểm tra | http://localhost:8000 |
| Platform | Docker Compose local fallback; chưa có phiên đăng nhập Railway/Render |
| Ngày kiểm tra | 2026-08-10 |
| Chế độ | `LOCAL_FALLBACK=true` |

Không có Railway/Render CLI, token hoặc phiên đăng nhập cloud trên máy thực
hiện bài, nên checkpoint sử dụng phương án local fallback chính thức. Stack
vẫn chạy từ production Dockerfile và kết nối tới Redis service qua Docker
Compose; không khai báo một URL cloud giả.

## Biến Môi Trường

Chỉ tên và nguồn của biến được ghi lại; giá trị khóa bí mật không nằm trong
tài liệu hoặc repository.

| Biến | Trạng thái | Nguồn |
|------|------------|-------|
| `PORT` | Đã set | Docker Compose, cổng 8000 |
| `AGENT_API_KEY` | Đã set | File `.env` không được commit |
| `REDIS_URL` | Đã set | `redis://redis:6379/0` trong mạng Compose |
| `RATE_LIMIT_PER_MINUTE` | Đã set | Cấu hình local |
| `MONTHLY_BUDGET_USD` | Đã set | Cấu hình local |
| `LOG_LEVEL` | Đã set | Cấu hình local |

## Kết Quả Chạy Thật

Khởi động stack:

```text
docker compose up -d --build
Image day12-2a202601145-leanhtien-agent Built
Container day12-2a202601145-leanhtien-redis-1 Healthy
Container day12-2a202601145-leanhtien-agent-1 Started
```

Trạng thái container:

```text
agent  Up (healthy)  0.0.0.0:8000->8000/tcp
redis  Up (healthy)  0.0.0.0:6379->6379/tcp
```

Kiểm tra endpoint:

```text
GET  /health:          HTTP 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready:           HTTP 200 {"status":"ready","redis":true}
POST /ask without key: HTTP 401 {"detail":"invalid or missing API key"}
POST /ask with key:    HTTP 200 answer_present=True user_id=cp5-auth-verification
```

Kiểm tra rate limit bằng một user riêng:

```text
200 200 200 200 200 200 200 200 200 200 429
```

## Ảnh Chụp

- `screenshots/local-fallback.png` — trạng thái stack và kết quả endpoint.
