# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: mỗi câu bên dưới đã được điền bằng quan sát và diễn giải của người làm bài.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Ngọc Thuận  Mã học viên: 2A202601949

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Ta chạy `Settings(_env_file=None)` sau khi xóa `AGENT_API_KEY` khỏi môi trường và nhận được `ValidationError: agent_api_key — Field required`. Đây là fail-fast đúng: app dừng trước khi phục vụ request; nếu dùng mặc định `changeme`, container vẫn chạy và có thể bị gọi trái phép.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")` không làm được.

Log thực tế thu được là `{"timestamp":"2026-08-10T04:12:27.143144+00:00","level":"info","event":"ask_completed","user_id":"exercise-log","tokens_in":4,"tokens_out":36,"cost_usd":2.22e-05}`. Từ đó có thể lọc request theo user và theo dõi token/chi phí; `print` thường chỉ là chuỗi tự do nên khó truy vấn theo từng trường.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản                 | Dung lượng |
| -------------------- | ------------ |
| 1 stage (bản đầu) | 1.73 GB      |
| Multi-stage          | 270 MB       |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Đo được 1.73 GB cho một stage và 270 MB cho multi-stage. Multi-stage nhỏ hơn vì chỉ chép dependency cần thiết và source vào runtime slim; compiler, cache và file tạm không đi vào stage cuối.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt `COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Trong log build thực tế, `COPY requirements.txt`, `pip install`, `COPY app/` và `COPY utils/` đều hiện `CACHED` vì chưa có file source nào thay đổi. Nếu sửa một dòng trong `app/main.py`, chỉ các layer copy source phía sau bị invalidated; nếu đặt `COPY . .` trước `pip install` thì pip install cũng phải chạy lại dù requirements không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Nếu app có lỗ hổng và container chạy root, kẻ tấn công có thể đọc/sửa nhiều file trong container và khai thác quyền cao hơn khi thoát container. Lệnh `docker image inspect agent:multi --format "User={{.Config.User}}"` cho kết quả `User=appuser`, xác nhận tiến trình chạy bằng tài khoản không đặc quyền.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được con số đó.

Với fixed window 10 request/phút, người dùng có thể gửi 10 request ở giây 59 của phút hiện tại rồi 10 request ở giây 00 của phút kế tiếp: tối đa 20 request trong khoảng 2 giây. Khi chạy thử sliding window, request 1–10 trả 200 còn request 11–12 trả 429; cửa sổ trượt không có khoảng reset này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit bảo vệ số lượng request và hạ tầng, còn cost guard bảo vệ số tiền. Test CP3 chạy thực tế đạt `22 passed`. Ví dụ rate limit cho qua nhưng cost guard chặn là user gửi mỗi phút một tài liệu rất dài; ngược lại, user còn ngân sách nhưng bắn 20 request trong vài giây sẽ bị rate limit chặn trước khi gọi LLM.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm 3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Nếu gộp health với kiểm tra Redis, Redis mất kết nối sẽ làm cả ba container trả lỗi; load balancer tưởng container chết và restart chúng. Thực tế khi dừng Redis, `/health = 200` còn `/ready = 503`; tách endpoint giúp health vẫn phản ánh process còn sống, còn ready ngắt traffic tạm thời.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một `X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Với dict trong mỗi process, container A và B có bộ nhớ riêng. Khi chạy 3 replica, 5 request thực tế nhận `history_length = 0, 2, 4, 6, 8`; nếu dùng dict, lịch sử sẽ bị phân mảnh theo replica và không tăng đều. Redis dùng chung giúp mọi request thấy cùng một lịch sử.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Khi thử lệnh kiểm tra cloud, tôi còn dùng URL mẫu `https://<domain-cua-ban>.up.railway.app`; curl báo `URL rejected: Bad hostname`. Nguyên nhân là chưa thay placeholder bằng domain Railway thật. Vì vậy các test cloud bị skip/chưa thể chạy; cần deploy thật, lấy domain HTTPS, cập nhật `DEPLOYMENT.md` rồi chạy lại CP5.
