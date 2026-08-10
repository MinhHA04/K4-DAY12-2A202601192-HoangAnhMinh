# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị token vào đây.**
> Repo này công khai — dán token vào là mất token.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Hoàng Anh Minh |
| Mã học viên | 2A202601192 |
| Repo | https://github.com/MinhHA04/K4-DAY12-2A202601192-HoangAnhMinh |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-chat-m9u9.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `API_TOKEN` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value `day12-chat-redis` |
| `BUCKET_CAPACITY` | ✅ | 10 |
| `REFILL_PER_MINUTE` | ✅ | 10 |
| `DAILY_BUDGET_USD` | ✅ | 1.0 |
| `LOG_LEVEL` | ✅ | INFO |

`REDIS_URL` trên Render do Blueprint lấy tự động từ private connection string
của `day12-chat-redis`. Không dùng giá trị local `redis://localhost:6379/0` trên
Render; `/readyz` trả 200 là bằng chứng kết nối hiện tại đã đúng.

## GitHub Actions CI/CD

Workflow: `.github/workflows/ci.yml`

Repository cần cấu hình tại **Settings → Secrets and variables → Actions**:

| Loại | Tên | Giá trị/nguồn |
|------|-----|---------------|
| Secret | `RENDER_DEPLOY_HOOK_URL` | Render service → Settings → Deploy Hook |
| Variable | `PUBLIC_URL` | `https://day12-chat-m9u9.onrender.com` |

Workflow chạy test và build Docker trên cả push lẫn pull request. Job deploy chỉ
chạy khi push lên `main`, sau khi hai job trước đã pass, rồi kiểm tra lại
`$PUBLIC_URL/healthz`. Không ghi Deploy Hook URL trực tiếp vào repository.

## Lệnh Kiểm Tra

Public URL được kiểm tra: `https://day12-chat-m9u9.onrender.com`.

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-chat-m9u9.onrender.com/healthz

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-chat-m9u9.onrender.com/readyz

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
curl -i -X POST https://day12-chat-m9u9.onrender.com/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

# 4. Có token — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-chat-m9u9.onrender.com/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-chat-m9u9.onrender.com/chat \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -H "X-Client-Id: sv-test" \
    -d '{"message":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
GET /healthz -> 200 {"status":"ok","service":"day12-chat-service","version":"1.0.0"}
GET /readyz  -> 200 {"status":"ready","redis":true}
POST /chat (không token) -> 401, header WWW-Authenticate: Bearer
POST /chat (Bearer token local) -> 200, trả về reply và usage
```

### Kết luận lần kiểm tra 2026-08-10

- Web service đã deploy thành công: `/healthz` trả 200.
- Render Key Value đã kết nối đúng: `/readyz` trả 200 và `redis=true`.
- `/chat` với `DEPLOY_API_TOKEN` local trả 200, chứng minh `API_TOKEN` trên
  Render đã được đồng bộ đúng.
- Bản deploy cũ trả 404 tại `/` vì hợp đồng ban đầu không khai báo route gốc.
  Source hiện đã bổ sung `GET /`; cần deploy phiên bản mới để mở domain trực
  tiếp và nhận trang thông tin API thay vì `{"detail":"Not Found"}`.

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/healthz.png` — kết quả gọi `/healthz` từ trình duyệt hoặc curl
