# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị token vào đây.**
> Repo này công khai — dán token vào là mất token.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Bùi Thái Sơn |
| Mã học viên | 2A202601126 |
| Repo | https://github.com/gliderest/K4_Day12_2A202601126_BuiThaiSon.git |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-chat-50r3.onrender.com/ |
| Platform | Render |
| Ngày deploy | 10-08-2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `API_TOKEN` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của platform / Upstash / ... |
| `BUCKET_CAPACITY` | ✅ | 10 |
| `REFILL_PER_MINUTE` | ✅ | 10 |
| `DAILY_BUDGET_USD` | ✅ | 1.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/healthz

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/readyz

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
curl -i -X POST <URL>/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

# 4. Có token — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/chat \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -H "X-Client-Id: sv-test" \
    -d '{"message":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

``` bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
HTTP/1.1 200 OK          
Date: Mon, 10 Aug 2026 09:42:41 GMT   
Content-Type: application/json   
Transfer-Encoding: chunked   
Connection: keep-alive   
rndr-id: 1ed082d6-a027-490c
Server: cloudflare            
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC          
CF-RAY: a28e0d4c8f90861d-HKG   
alt-svc: h3=":443"; ma=86400   
   
{"status":"ok","service":"day12-chat-service","version":"1.0.0"}

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
HTTP/1.1 200 OK
Date: Mon, 10 Aug 2026 09:43:54 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 437fe3ba-bb48-48c7
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a28e0f167c7bfdac-SIN
alt-svc: h3=":443"; ma=86400

{"status":"ready","redis":true}

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
HTTP/1.1 401 Unauthorized
Date: Mon, 10 Aug 2026 09:48:08 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: 6df9c91e-a642-40b1
Server: cloudflare
vary: Accept-Encoding
www-authenticate: Bearer
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a28e15446c3ae2ed-HKG
alt-svc: h3=":443"; ma=86400

{"detail":"invalid or missing bearer token"}curl: (3) URL rejected: Bad hostname

# 4. Có token — mong đợi 200 kèm câu trả lời
HTTP/1.1 200 OK
Date: Mon, 10 Aug 2026 09:57:34 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: fce58490-dd12-4daa
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a28e2315ace3c4d2-SIN
alt-svc: h3=":443"; ma=86400

{"reply":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","client_id":"sv-test","turns_before":0,"usd_cost":2.145e-05,"usage":{"prompt":3,"completion":35}}

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
200 200 200 200 200 200 200 200 200 200 429 429 429 200 429 

```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/healthz.png` — kết quả gọi `/healthz` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```

```
