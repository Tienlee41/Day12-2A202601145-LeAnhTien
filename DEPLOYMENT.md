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
| Public URL | https://day12-agent-l002.onrender.com |
| Platform | Render Blueprint, Docker Web Service và Render Key Value |
| Branch | `main` |
| Commit đã deploy | `6cf23d1` (`cp5`) |
| Ngày kiểm tra | 2026-08-10 |

Render build production `Dockerfile`, cấp HTTPS cho web service và kết nối
service với `day12-redis` qua mạng nội bộ. Endpoint gốc `/` không được khai báo;
các endpoint vận hành chính thức là `/health`, `/ready`, `/ask` và `/docs`.

## Biến Môi Trường Trên Render

Chỉ tên và nguồn của biến được ghi lại; giá trị khóa bí mật không nằm trong
tài liệu hoặc repository.

| Biến | Trạng thái | Nguồn |
|------|------------|-------|
| `PORT` | Đã set | Render tự cấp; ứng dụng đọc biến lúc khởi động |
| `AGENT_API_KEY` | Đã set | Render Environment, giá trị được ẩn |
| `REDIS_URL` | Đã set | Internal Key Value URL của `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | Đã set | Render Blueprint, giá trị 10 |
| `MONTHLY_BUDGET_USD` | Đã set | Render Blueprint, giá trị 10.0 |
| `LOG_LEVEL` | Đã set | Render Blueprint, giá trị INFO |

## Kết Quả Kiểm Tra Cloud

```text
GET  /health:          HTTP 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready:           HTTP 200 {"status":"ready","redis":true}
POST /ask without key: HTTP 401 {"detail":"invalid or missing API key"}
POST /ask with key:    HTTP 200 answer_present=True user_id=cp5-final-auth
```

Kiểm tra rate limit bằng một user riêng:

```text
200 200 200 200 200 200 200 200 200 200 429
```

Các kết quả trên chứng minh web process hoạt động, Redis nội bộ sẵn sàng,
endpoint có xác thực chặn request không key, request hợp lệ nhận được câu trả
lời và hạn mức 10 request/phút được thực thi.

## Ảnh Kiểm Tra

- `screenshots/render-verification.png` — kết quả gọi service Render công khai.
