# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng placeholder mẫu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Tiến Thành  Mã học viên: 2A202601539

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu deploy lên Railway/Render mà quên set `AGENT_API_KEY`, app sẽ lỗi ngay lúc khởi động và log báo thiếu biến môi trường. Nhờ vậy mình biết phải cấu hình secret trước khi public service. Nếu để mặc định `"changeme"`, service vẫn chạy, người khác có thể đoán hoặc dùng khóa mặc định để gọi API, làm lộ endpoint và phát sinh chi phí mà mình không phát hiện ngay.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log JSON mình thu được:
> `{"event": "ask_completed", "level": "info", "timestamp": "2026-08-10T03:25:09.538838+00:00", "user_id": "sv01", "tokens_in": 12, "tokens_out": 24, "cost_usd": 0.0001}`
>
> Với log dạng này mình có thể lọc theo `event`, `level`, `user_id` để biết user nào gọi nhiều hoặc request nào lỗi. Mình cũng có thể cộng `cost_usd`, `tokens_in`, `tokens_out` để theo dõi chi phí/token theo thời gian. Một dòng `print("đã trả lời xong")` không có cấu trúc nên máy khó lọc, khó gom nhóm và khó cảnh báo tự động.

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
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần chênh lệch chủ yếu đến từ base image `python:3.11` bản đầy đủ nặng hơn `python:3.11-slim`, cache/tạm của `pip install` trong bản cũ, và việc `COPY . .` mang nhiều file không cần thiết vào image. Bản multi-stage chỉ copy phần dependency đã cài từ stage `builder` sang runtime và chỉ copy `app`, `utils`, nên image cuối nhỏ hơn nhiều.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Với Dockerfile hiện tại, khi chỉ sửa một ký tự trong `app/main.py`, các layer cài dependency vẫn dùng cache: `COPY requirements.txt` và `RUN pip install` không chạy lại vì `requirements.txt` không đổi. Phần phải chạy lại bắt đầu từ `COPY app ./app` và các layer phía sau nó. Nếu đặt `COPY . .` trước `RUN pip install`, mỗi lần sửa source code Docker sẽ invalid cache từ bước copy source, làm `pip install` chạy lại dù dependency không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python có lỗ hổng cho phép chạy lệnh trong container, attacker sẽ có quyền của user đang chạy process. Nếu process chạy bằng root, các lệnh đó chạy với quyền root trong container; khi kết hợp với mount sai, capability nguy hiểm hoặc lỗi runtime/container escape, rủi ro leo thang lên host sẽ lớn hơn. Lệnh `USER appuser` cắt chuỗi này ở chỗ process không còn chạy bằng root, nên kể cả khi app bị khai thác thì attacker chỉ có quyền user thường trong container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Nếu đếm theo phút đồng hồ và hạn mức là 10 request/phút, user có thể gửi tối đa 20 request trong khoảng 2 giây: gửi 10 request vào lúc 10:00:59, sau đó sang 10:01:00 bộ đếm reset và gửi tiếp 10 request nữa vào 10:01:01. Cả hai phút đều không vượt 10 request, nhưng thực tế trong 2 giây hệ thống nhận 20 request. Sliding window 60 giây chặn được vì nó nhìn 60 giây gần nhất, không phụ thuộc mốc giây 00.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tốc độ/số lượng request trong một khoảng thời gian, còn cost guard giới hạn tổng chi phí theo ngân sách tháng. Trường hợp rate limit cho qua nhưng cost guard chặn: user chỉ gửi 1 request/phút nhưng đã tiêu gần hết ngân sách tháng, request mới có cost ước tính làm vượt budget nên phải trả 402. Trường hợp ngược lại: user còn rất nhiều ngân sách nhưng gửi quá nhanh, ví dụ request thứ 11 trong cùng cửa sổ 60 giây khi limit là 10/phút, nên rate limit trả 429 dù cost guard vẫn còn cho phép.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu `/health` cũng kiểm tra Redis, khi Redis mất kết nối 30 giây thì cả 3 container đều bắt đầu trả 503 cho health check. Orchestrator hiểu nhầm là process bị lỗi chứ không phải dependency bị lỗi, nên restart từng container hoặc cả cụm. Trong lúc Redis chưa quay lại, container mới khởi động vẫn health check fail tiếp, dẫn tới vòng lặp restart và không còn instance ổn định để phục vụ. Đúng ra `/health` chỉ báo process sống hay không, còn `/ready` mới báo Redis chết để load balancer tạm ngừng gửi traffic vào instance.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu lưu trong dict Python, mỗi container sẽ có một bộ nhớ riêng. Khi load balancer chuyển các request của cùng `X-User-Id` qua các container khác nhau, `history_length` sẽ tăng không đều hoặc quay về 0/2 tùy request rơi vào instance nào. Ví dụ câu đầu vào container A, câu sau vào container B thì B không thấy lịch sử của A. Khi lưu trong Redis, cả 3 container cùng đọc/ghi một nơi nên các request sau nhìn thấy lịch sử trước đó ổn định hơn.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi làm CP5 trên máy local, mình chưa deploy được cloud vì lệnh `railway --version` báo `railway : The term 'railway' is not recognized...`, nghĩa là máy chưa có Railway CLI trong PATH và cũng chưa có session đăng nhập Railway/Render để tạo public URL. Mình kiểm tra nguyên nhân bằng terminal, sau đó dùng phương án dự phòng của lab: đặt `LOCAL_FALLBACK=true`, chạy `docker compose up -d --build`, kiểm tra `/health` trả 200, `/ready` trả 200 và `/ask` không có API key trả 401. Nếu deploy thật, bước cần làm tiếp là đăng nhập Railway/Render, set `AGENT_API_KEY` và `REDIS_URL` trên dashboard rồi điền Public URL thật vào `DEPLOYMENT.md`.
