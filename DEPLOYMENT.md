# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. Không ghi giá trị API key vào repository.

## Thông Tin Học Viên

| Mục           | Nội dung                         |
| -------------- | --------------------------------- |
| Họ và tên   | Nguyen Ngoc Thuan                 |
| Mã học viên | 2A202601949                       |
| Repo           | DAY12-2A202601949-NguyenNgocThuan |

## Service

| Mục         | Nội dung                                                    |
| ------------ | ------------------------------------------------------------ |
| Public URL   | https://TODO-thay-bang-url-that.up.railway.app               |
| Platform     | Railway / Render / Cloud Run — (điền platform bạn dùng) |
| Ngày deploy | (điền ngày)                                               |

## Biến Môi Trường Đã Set Trên Cloud

| Biến                     | Đã set | Ghi chú                                      |
| ------------------------- | -------- | --------------------------------------------- |
| `PORT`                  | ✅       | platform tự gán                             |
| `AGENT_API_KEY`         | ✅       | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL`             | ✅       | Redis add-on của platform                    |
| `RATE_LIMIT_PER_MINUTE` | ✅       | 10                                            |
| `MONTHLY_BUDGET_USD`    | ✅       | 10.0                                          |
| `LOG_LEVEL`             | ✅       | INFO                                          |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
(điền output)
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

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
(điền lý do nếu dùng phương án dự phòng, ngược lại xóa mục này)
```
