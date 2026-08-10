# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng giữ chỗ dưới mỗi câu hỏi bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Hoàng Anh Minh  Mã học viên: 2A202601192

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ khi deploy lên Render, nếu tôi quên đặt `API_TOKEN` thì ứng dụng dừng ngay và log báo thiếu biến môi trường. Nhờ vậy tôi biết lỗi để sửa trước khi service nhận request. Nếu dùng mặc định `"changeme"`, ứng dụng vẫn chạy và người lạ có thể đoán token này để gọi API.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi thu được:
>
> `{"event": "chat_completed", "severity": "INFO", "ts": "2026-08-10T13:26:02.016196+00:00", "client_id": "sv-exercise", "prompt_tokens": 5, "completion_tokens": 37, "usd_cost": 2.295e-05}`
>
> Từ log này, tôi có thể lọc các sự kiện `chat_completed` hoặc các log theo mức `severity`. Tôi cũng có thể cộng `usd_cost` theo `client_id` để biết client nào dùng nhiều tiền nhất. Dòng `print("đã trả lời xong")` không có các trường dữ liệu này nên không lọc hay tính toán được.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 431,6 MB |
| Multi-stage | 63,7 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi đo bằng `docker image inspect`: bản một stage là 431.585.516 byte, còn bản multi-stage là 63.721.348 byte. Phần chênh lệch chủ yếu đến từ image `python:3.11` đầy đủ có nhiều gói hệ điều hành và công cụ không cần lúc chạy. Bản mới dùng `python:3.11-slim` và chỉ chép thư viện đã cài từ builder sang runtime, nên không mang theo môi trường build nặng.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, các layer lấy base image, `COPY requirements.txt` và `pip install` vẫn được dùng lại từ cache vì dependency không đổi. Layer `COPY app ./app` và các layer đứng sau nó phải tạo lại. Nếu đặt `COPY . .` trước `pip install`, chỉ cần sửa một dòng code cũng làm layer copy thay đổi, khiến Docker phải cài lại toàn bộ thư viện và build lâu hơn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng có lỗ hổng cho phép chạy lệnh, kẻ tấn công có thể chạy lệnh bên trong container. Container chạy bằng root thì các lệnh đó cũng có quyền root, nếu tiếp tục khai thác được lỗi của runtime hoặc cấu hình mount nguy hiểm, kẻ tấn công có thể ảnh hưởng tới máy host. `USER appuser` chuyển process sang UID 10001 có quyền thấp, nên dù chiếm được ứng dụng thì kẻ tấn công cũng không có sẵn quyền root trong container.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> Header `WWW-Authenticate: Bearer` cho client biết API yêu cầu xác thực bằng Bearer token, đúng quy ước của HTTP. Ta dùng cùng thông báo cho trường hợp thiếu header, sai scheme và sai token để người đang dò token không biết họ đã làm đúng phần nào. Client hợp lệ vẫn biết cách sửa nhờ tài liệu API.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Có `min(capacity, ...)`, xô không bao giờ chứa quá 10 token nên client gửi được 10 request liên tiếp; request thứ 11 nhận 429. Nếu bỏ `min`, trong 10 phút xô có thể nhận thêm 100 token. Nếu trước đó xô đang đầy 10 token thì nó có khoảng 110 token và có thể gửi khoảng 110 request. Đây là lý do phải chặn số token tối đa ở `capacity`.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Với hạn mức 30 USD/tháng, sự cố có thể tiêu gần hết 30 USD ngay trong một đêm và client chỉ tự dùng lại được khi sang tháng mới. Với hạn mức 1 USD/ngày, thiệt hại trong ngày được giới hạn khoảng 1 USD; sang ngày UTC mới, key chi phí đổi theo ngày nên service tự cho client hoạt động lại. Hạn mức ngày vì vậy giới hạn thiệt hại nhỏ hơn và phục hồi nhanh hơn.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Thứ tự xảy ra là: Redis mất kết nối; cả 3 container cùng kiểm tra Redis và báo endpoint chung là 503; orchestrator hiểu nhầm rằng cả 3 process đã hỏng nên restart chúng cùng lúc; trong lúc restart không còn container phục vụ và người dùng nhận lỗi. Khi Redis hoạt động lại, các container vẫn cần thời gian khởi động. Tách `/healthz` và `/readyz` giúp load balancer chỉ ngừng gửi traffic khi Redis lỗi, thay vì restart toàn bộ container.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi tôi gặp là `/healthz` và `/readyz` đều trả 200 nhưng gọi `/chat` bằng token trong `.env` lại trả `401 {"detail":"invalid or missing bearer token"}`. Tôi kiểm tra và thấy `API_TOKEN` với `DEPLOY_API_TOKEN` ở local giống nhau, nên nguyên nhân là `API_TOKEN` đang chạy trên Render vẫn là giá trị cũ. `sync: false` chỉ hỏi secret khi tạo Blueprint lần đầu, không tự đồng bộ file `.env`. Cách sửa là vào Render -> Environment, cập nhật `API_TOKEN` bằng đúng token local, chọn Save Changes và chờ service deploy lại. `REDIS_URL` không phải nguyên nhân vì `/readyz` đã trả 200.
