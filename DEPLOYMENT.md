# Thông Tin Deploy — Checkpoint 5

> Service đã được deploy bằng Render Blueprint từ `render.yaml`.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không ghi giá trị API key.**

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Trần Văn Dũng |
| Mã học viên | 2A202601859 |
| Repo hiện tại | https://github.com/vdungx/DAY12_2A20261859_TranVanDung |
| Tên bài nộp yêu cầu | `K3-DAY12-2A202601859-TranVanDung` |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-kh0x.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | Secret đặt trong Render Dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value internal connection string |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

```powershell
curl.exe -i https://day12-agent-kh0x.onrender.com/health
curl.exe -i https://day12-agent-kh0x.onrender.com/ready

'{"question":"Hello"}' |
  curl.exe -i -X POST `
    "https://day12-agent-kh0x.onrender.com/ask" `
    -H "Content-Type: application/json" `
    --data-binary "@-"

'{"question":"Deploy la gi?"}' |
  curl.exe -i -X POST `
    "https://day12-agent-kh0x.onrender.com/ask" `
    -H "Content-Type: application/json" `
    -H "X-API-Key: $env:DEPLOY_API_KEY" `
    -H "X-User-Id: cp5-live-test" `
    --data-binary "@-"
```

## Kết Quả Chạy Thật

```text
GET  /health                  -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready                   -> 200 {"status":"ready","redis":true}
POST /ask (không API key)     -> 401 {"detail":"invalid or missing API key"}
POST /ask (API key hợp lệ)    -> 200, user_id=cp5-live-test, history_length=0, cost_usd=2.265e-05
```

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — Render Dashboard hiển thị deploy thành công.
- `screenshots/health.png` — `/health` trả 200.
- `screenshots/ready.png` — `/ready` trả 200 và Redis sẵn sàng.
- `screenshots/ask401.png` — `/ask` không API key trả 401.
- `screenshots/ask200.png` — `/ask` có API key hợp lệ trả 200.

Các ảnh không hiển thị giá trị API key.

## Phương Án Dự Phòng

Không dùng phương án dự phòng; service đã deploy thành công trên Render.
