# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng placeholder bên dưới bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trần Văn Dũng  Mã học viên: 2A202601859

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Trong lần deploy đầu tiên lên Render, process thoát với status 3 vì chưa cấu hình `AGENT_API_KEY`. Fail-fast giúp tôi phát hiện cấu hình cloud còn thiếu trước khi service nhận request. Nếu dùng mặc định `"changeme"`, service có thể báo deploy thành công nhưng chạy với key giả dễ đoán, khiến tôi tưởng API đã an toàn.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi thu được là `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T04:29:14.420598+00:00","user_id":"exercise-log","tokens_in":4,"tokens_out":38,"cost_usd":0.0000234}`. Với log này, tôi có thể lọc theo `user_id` hoặc `level`, đồng thời tổng hợp token và chi phí để làm dashboard/cảnh báo. `print("đã trả lời xong")` không có các trường ổn định để máy tìm kiếm hoặc tính toán.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 446.1 MB |
| Multi-stage | 63.7 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi đo được image 1-stage là **446.1 MB**, còn image multi-stage là **63.7 MB**. Image multi-stage nhỏ hơn khoảng 382.4 MB. Chênh lệch chủ yếu đến từ base image Python đầy đủ, các thành phần hệ điều hành và dữ liệu phục vụ build; bản production dùng image slim và chỉ copy package đã cài từ builder sang runtime.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, các layer base image, `WORKDIR`, copy `requirements.txt` và cài dependency vẫn dùng cache; Docker chỉ chạy lại layer copy source phía sau. Nếu đặt `COPY . .` trước `RUN pip install`, mọi thay đổi source sẽ làm layer copy đổi và buộc cài lại package dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng Python có lỗi cho phép thực thi lệnh, kẻ tấn công chạy lệnh với quyền của process container. Khi process là root, họ có thể sửa file hệ thống, cài công cụ hoặc lợi dụng mount/runtime để tìm đường leo thang sang host. `USER appuser` giới hạn process bị chiếm quyền ở user thường, cắt chuỗi tấn công tại bước giành quyền root trong container và giảm thiệt hại.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với fixed window 10 request/phút, user có thể gửi 10 request ở giây 59 rồi thêm 10 request ở giây 00 của phút kế tiếp. Như vậy tối đa là **20 request trong khoảng 2 giây** mà mỗi cửa sổ vẫn không vượt 10. Sliding window nhìn lại đúng 60 giây nên loạt trước giây 00 vẫn được tính và loạt thứ hai sẽ bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit bảo vệ tốc độ và tài nguyên hạ tầng trong 60 giây, còn cost guard bảo vệ ngân sách LLM tích lũy theo tháng. Nếu user gửi 1 request/phút nhưng mỗi request xử lý nội dung rất dài, rate limit cho qua còn cost guard chặn khi hết budget. Ngược lại, nếu user gửi 11 câu hỏi rất ngắn liên tiếp khi ngân sách còn nhiều, cost guard cho qua về chi phí nhưng request thứ 11 bị rate limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp hai endpoint và probe chung kiểm tra Redis, khi Redis mất kết nối thì cả ba container đều báo lỗi. Load balancer rút chúng khỏi danh sách nhận traffic; orchestrator có thể coi process hỏng và restart chúng. Container mới vẫn không nối được Redis nên tiếp tục fail và restart, tạo vòng lặp trong 30 giây. Khi tách probe, `/ready` trả 503 để ngắt traffic còn `/health` vẫn cho biết process sống, tránh restart oan vì dependency lỗi tạm thời.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi chạy ba replica và gọi qua Nginx với cùng `X-User-Id`, tôi thấy `history_length` tăng `0, 2, 4, 6, 8`; mỗi lượt thêm một message user và một message assistant vào Redis chung. Nếu dùng dict Python, mỗi replica chỉ thấy lịch sử riêng nên số sẽ lặp hoặc nhảy theo replica, ví dụ các request đầu cùng trả 0; restart container còn làm mất lịch sử cục bộ.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần deploy đầu trên Render báo `Exited with status 3`. Tôi đọc deploy log và chạy lại lệnh khởi động trong môi trường thiếu key, thấy Pydantic báo thiếu `AGENT_API_KEY`. Nguyên nhân là secret chưa được khai báo cho web service. Tôi thêm biến này trong Environment của Render rồi redeploy; sau đó `/health`, `/ready` trả 200, `/ask` không key trả 401 và có đúng key trả 200.
