# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng trả lời mẫu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Lê Anh Tiến  Mã học viên: 2A202601145

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Render, lúc đầu tôi chưa đặt `AGENT_API_KEY`: `/health` vẫn
> sống nhưng các endpoint cần cấu hình trả lỗi 500. Việc secret là trường bắt
> buộc giúp lỗi lộ ra ngay trong lần kiểm tra deploy thay vì âm thầm chạy với
> khóa `"changeme"`. Nếu có khóa mặc định, bot quét Internet có thể đoán khóa,
> gọi `/ask` và tiêu ngân sách trong khi dashboard vẫn báo service hoạt động.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log thật tôi thu được là:
> `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T05:55:06.372281+00:00","user_id":"exercise-log","tokens_in":7,"tokens_out":41,"cost_usd":2.565e-05}`.
> Với JSON này tôi có thể (1) lọc theo `user_id` rồi cộng `cost_usd` để biết ai
> tiêu nhiều nhất, và (2) nhóm theo `event`, `level`, khoảng thời gian để dựng
> dashboard hoặc cảnh báo tỷ lệ lỗi. Chuỗi `print("đã trả lời xong")` không có
> các trường có cấu trúc để máy thực hiện hai phép phân tích đó.

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
| 1 stage (bản đầu) | 1,73 GB |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi build lại Dockerfile một-stage từ commit trước CP2 và đo bằng
> `docker images`: bản cũ 1,73 GB, bản multi-stage 270 MB, giảm khoảng 1,46 GB.
> Phần chênh lệch chủ yếu là base image Python/Debian đầy đủ và các thành phần
> chỉ cần trong quá trình cài đặt. Bản mới dùng `python:3.11-slim`; stage
> builder cài dependency rồi runtime chỉ nhận kết quả `/install`, nên không
> mang toàn bộ môi trường build sang image cuối.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ đổi `app/main.py`, lần build lại cho thấy các layer copy
> `requirements.txt`, `pip install`, tạo `appuser`, `WORKDIR` và copy dependency
> từ builder đều dùng cache. Layer `COPY app ./app` và các layer phía sau nó
> phải tạo lại. Nếu đặt `COPY . .` trước `RUN pip install`, thay đổi một ký tự
> trong source sẽ làm layer copy mất cache, kéo theo layer cài toàn bộ package
> chạy lại; thời gian build tăng dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Một lỗi RCE trong API có thể cho kẻ tấn công chạy lệnh với UID của process.
> Nếu process là root, chúng có toàn quyền trong container và có vị trí thuận
> lợi hơn để lợi dụng kernel/container runtime, volume nhạy cảm hoặc Docker
> socket cấu hình sai để leo sang host. `USER appuser` cắt chuỗi ngay sau bước
> RCE: mã độc chỉ có quyền của user thường, không sửa được file hệ thống hay
> thực hiện thao tác đặc quyền. Nó không thay thế sandbox nhưng giảm mạnh phạm
> vi thiệt hại nếu ứng dụng bị chiếm quyền.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa là 20 request trong khoảng 2 giây: gửi 10 request vào 10:00:59, bộ
> đếm theo phút reset lúc 10:01:00, rồi gửi tiếp 10 request ngay sau đó. Mỗi
> phút riêng lẻ vẫn đúng hạn mức 10 nhưng tải thực tế là 20 request gần như
> đồng thời. Sliding window nhìn lại đúng 60 giây nên loạt thứ hai bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong 60 giây, còn cost guard giới hạn tổng
> tiền của từng user trong tháng. Một user chỉ gửi một request rất dài, vẫn
> dưới 10 request/phút, nhưng đã tiêu gần hết 10 USD thì rate limit cho qua còn
> cost guard phải trả 402. Ngược lại, một user mới chưa tốn đáng kể nhưng gửi
> 11 câu hỏi rất ngắn trong một phút thì cost guard vẫn cho phép về ngân sách,
> còn rate limiter chặn request thứ 11 bằng 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp probe và cho nó ping Redis, khi Redis mất kết nối thì cả ba container
> cùng trả 503. Orchestrator hiểu đây là lỗi liveness nên rút traffic và restart
> cả ba gần như đồng thời. Các instance mới vẫn không ping được Redis, tiếp tục
> fail và có thể rơi vào vòng lặp restart; trong lúc đó không còn instance nào
> phục vụ dù process web vốn vẫn sống. Tách `/ready` chỉ làm load balancer ngừng
> gửi request, còn `/health` vẫn 200 nên container không bị restart vô ích.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với cùng `X-User-Id` và Redis dùng chung, `history_length` tăng đều theo số
> message đã lưu: 0, 2, 4, 6, 8... Mỗi lượt thêm một message user và một message
> assistant nên tăng 2, bất kể request rơi vào instance nào. Nếu dùng dict
> Python riêng trong từng container, mỗi instance có một lịch sử khác nhau;
> qua load balancer tôi sẽ thấy số bị lặp hoặc nhảy lùi như 0, 0, 2, 0, 2 thay
> vì một dãy tăng liên tục, và restart container còn làm lịch sử về 0.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi thực tế tôi gặp là `/health` trả 200 nhưng `/ready` trả 503 với
> `{"status":"not ready","redis":false}`. Vì liveness vẫn tốt nên tôi loại
> trừ lỗi container/port và tập trung vào dependency Redis. Trong Render
> Dashboard tôi phát hiện `day12-redis` đang ở trạng thái `Suspended` và web
> service chưa dùng đúng Internal Key Value URL. Tôi resume Key Value instance,
> sao chép Internal URL vào `REDIS_URL`, lưu và deploy lại `day12-agent`. Sau
> đó `/ready` trả 200 với `{"status":"ready","redis":true}`, `/ask` không key
> trả 401 và request có key trả 200.
