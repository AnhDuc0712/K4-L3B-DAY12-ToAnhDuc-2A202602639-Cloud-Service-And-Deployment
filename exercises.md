# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Tô Anh Đức  Mã học viên: 2A202602639

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> ví dụ khi deploy production quên cấu hình agent_api_key, app dừng ngay nên không vô tình chạy với khóa mặc định dễ đoán.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> {"event":"ask_completed","level":"info","timestamp":"2026-09-29T04:27:41.653644+00:00","user_id":"cp5-test","tokens_in":41,"tokens_out":45,"cost_usd":0.00003315}
Biết yêu cầu của cp5-test dùng 41 token đầu vào, 45 token đầu ra, nên có thể theo dõi mức sử dụng.
Biết thời điểm, mức độ (info) và chi phí cụ thể ($0.00003315) để kiểm toán và tính chi phí.

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
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần chênh khoảng 1.46 GB là hệ điều hành và bộ công cụ của image python:3.11 đầy đủ (compiler, header, gói hệ thống) cùng stage builder bị bỏ lại. Image cuối chỉ giữ python:3.11-slim và thư viện đã cài.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Đổi một ký tự trong app/main.py: cache dùng lại các layer cài dependencies; build lại từ COPY app ./app trở đi, gồm COPY utils và useradd.
Nếu COPY . . đứng trước RUN pip install, thay đổi code làm mất cache cài thư viện nên pip install phải chạy lại.


---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Lỗ hổng Python cho phép kẻ tấn công chạy lệnh trong container; nếu tiến trình là root, họ có quyền cao bên trong container và có thể khai thác cấu hình sai để thoát container, ảnh hưởng host.
USER appuser chạy app với quyền hạn chế, cắt chuỗi leo thang ngay tại bước chiếm quyền root trong container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

>Có thể gửi tối đa 20 request trong 2 giây.
Gửi 10 request ngay trước khi phút đổi, rồi 10 request ngay sau giây 00; bộ đếm theo phút đã reset dù chỉ cách nhau vài giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

>Rate limit giới hạn số request theo thời gian; cost guard giới hạn tổng chi phí đã tiêu.
Request thứ 11 trong một phút bị rate limit chặn dù còn ngân sách; request đắt tiền bị cost guard chặn dù vẫn dưới 10 request/phút.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

>Redis mất kết nối /health thất bại ở cả 3 container , orchestrator đánh dấu chúng không khỏe và khởi động lại/loại khỏi cụm.
Các container vẫn có thể xử lý request không cần Redis, nhưng bị loại cùng nhau; /ready riêng chỉ nên báo chưa sẵn sàng nhận traffic khi Redis không khả dụng.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Ba container nhận cùng X-User-Id lần lượt trả history_length là 0, rồi 2, rồi 4. Lịch sử nằm ở Redis nên container sau vẫn thấy tin nhắn của container trước. Nếu lưu trong một dict Python, mỗi container chỉ nhớ phần của chính nó, nên lần gọi sang container khác có thể lại ra 0 hoặc một số nhỏ hơn tổng số tin đã gửi.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần gọi /ask đầu tiên lên URL Railway trả 422 với thông báo JSON decode error: Expecting property name enclosed in double quotes. Nguyên nhân là PowerShell làm mất dấu ngoặc kép trong body, không phải Redis hay $PORT. Gửi lại đúng JSON thì không có key ra 401, có key ra 200
