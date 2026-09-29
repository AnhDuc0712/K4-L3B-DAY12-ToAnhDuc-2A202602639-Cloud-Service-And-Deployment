# Thông Tin Deploy — Checkpoint 5

> `pytest tests/test_cp5.py` đọc file này để tìm địa chỉ service và gọi thử.
>
> Chỉ ghi TÊN biến môi trường. Không dán giá trị API key.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Tô Anh Đức |
| Mã học viên | chưa lưu trong repo |
| Repo | https://github.com/AnhDuc0712/K4-L3B-Cloud-Service-And-Deployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://agent-production-e6bb.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và nguồn giá trị, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | không set tay | Railway tự gán, service không ghi đè |
| `AGENT_API_KEY` | có | Variables của service `agent`, không nằm trong repo |
| `REDIS_URL` | có | tham chiếu Redis add-on của project (`Redis.REDIS_URL`) |
| `RATE_LIMIT_PER_MINUTE` | có | 10 |
| `MONTHLY_BUDGET_USD` | có | 10.0 |
| `LOG_LEVEL` | có | INFO |

## Lệnh Kiểm Tra

```bash
URL=https://agent-production-e6bb.up.railway.app

curl -i $URL/health
curl -i $URL/ready
curl -i -X POST $URL/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
```

## Kết Quả Chạy Thật

```
GET /health
HTTP/1.1 200 OK
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP/1.1 200 OK
{"status":"ready","redis":true}

POST /ask  (không có API key)
HTTP/1.1 401 Unauthorized
{"detail":"invalid or missing API key"}

POST /ask  (có API key, header X-User-Id: sv-test)
HTTP/1.1 200 OK
user_id=sv-test, history_length=0, cost_usd=0.00002265

Rate limit, 15 lần, user sv-rate, hạn mức 10/phút:
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — project Railway `day12-agent` (service `agent` và Redis)
- `screenshots/health.png` — `/health` trên URL công khai

Bản này deploy trên Railway, không dùng phương án dự phòng.
