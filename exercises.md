# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.

**Họ và tên:** Bùi Thái Sơn  
**Mã học viên:** 2A202601126

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu để mặc định là `"changeme"`, khi deploy lên production mà tôi quên cấu hình biến môi trường, ứng dụng vẫn chạy bình thường. Hậu quả là hacker có thể dễ dàng đoán được token này, truy cập trái phép vào API và đánh cắp/xóa dữ liệu. Việc ứng dụng "chết sớm" (Fail fast) buộc tôi phải nhận ra lỗi cấu hình ngay lập tức ở giai đoạn khởi động (hoặc giai đoạn CI/CD) trước khi code kịp chạm ra môi trường thực.

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")` không làm được.

> **Dòng log JSON thu được (ví dụ):**
>
> `{"level": "INFO", "timestamp": "2026-08-11T10:28:34Z", "method": "POST", "path": "/chat", "latency_ms": 45.2, "status_code": 200, "client_ip": "127.0.0.1"}`
>
> **Hai việc làm được với log JSON:**
>
> 1. Dễ dàng truy vấn và lọc dữ liệu bằng các hệ thống quản lý log (như ELK, Datadog). Ví dụ: Lọc ra toàn bộ các request có `latency_ms` > 1000 để tìm API bị chậm.
> 2. Có thể trích xuất các trường dữ liệu (như `status_code`) để vẽ các biểu đồ theo dõi sức khỏe hệ thống (Dashboard) một cách tự động, điều mà lệnh `print` text thuần không thể làm được do máy tính khó bóc tách thông tin.

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

| **Bản** | **Dung lượng** |
|---|---:|
| 1 stage (bản đầu) | 856 MB |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch đó chính là: bộ công cụ dùng để build code (trình biên dịch, pip), các thư viện phát triển (dev-dependencies), mã nguồn gốc (source code) chưa biên dịch, và các file cache sinh ra trong quá trình cài đặt. Multi-stage build loại bỏ toàn bộ "công xưởng" này, chỉ copy file thực thi hoặc môi trường runtime tối thiểu cuối cùng sang stage đích.

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt `COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> - **Khi sửa `main.py`:** Các layer trước lệnh `COPY ./app` (như kéo base image, `COPY requirements.txt`, `RUN pip install`) sẽ được dùng lại từ **cache**. Từ layer `COPY ./app` trở đi sẽ phải **chạy lại**.
> - **Nếu đặt `COPY . .` lên trước `RUN pip install`:** Bất kỳ thay đổi nhỏ nào trong code (dù chỉ là sửa 1 ký tự trong file `main.py`) cũng sẽ làm vô hiệu hóa bộ nhớ cache của lệnh `COPY`. Hậu quả là Docker sẽ phải chạy lại lệnh `RUN pip install` và tải lại toàn bộ thư viện từ đầu, làm tăng đáng kể thời gian build image.

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> **Chuỗi sự kiện:** Có lỗ hổng thực thi mã từ xa (RCE) trong code Python -> Kẻ tấn công gọi API để chạy mã độc -> Do app chạy bằng root, mã độc có quyền root bên trong container -> Nếu container bị cấu hình lỏng lẻo (bind mount thư mục host, chạy privileged), kẻ tấn công dùng quyền root này để thao túng hoặc thoát ra ngoài chiếm quyền kiểm soát máy host.
>
> **Lệnh `USER` cắt đứt ở đâu:** Lệnh `USER appuser` chuyển quyền chạy tiến trình app sang một user thường (không có quyền root). Khi kẻ tấn công khai thác được RCE, chúng chỉ lấy được vỏ bọc (shell) của một user bị giới hạn quyền. Chúng không thể cài thêm phần mềm độc hại, không thể sửa các file hệ thống, và mất đi bàn đạp để leo thang đặc quyền ra máy host.

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token) thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> - **Vì sao cần header:** Đây là quy chuẩn của giao thức HTTP (RFC 6750). Nó báo cho client (hoặc trình duyệt) biết hệ thống này yêu cầu phương thức xác thực nào (ở đây là Bearer token) để client biết cách cung cấp cho đúng.
> - **Vì sao thông báo lỗi chung chung:** Để đảm bảo bảo mật (chống rò rỉ thông tin). Nếu ta nói rõ "Token đúng nhưng hết hạn" hoặc "Scheme sai", kẻ tấn công có thể dựa vào đó để dò dẫm, thu hẹp phạm vi đoán hoặc brute-force token. Trả lời chung chung giúp ẩn đi logic kiểm tra bên trong hệ thống.

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn `min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> - Gửi được tối đa **10** request trước khi bị 429.
> - Nếu bỏ đoạn `min(capacity, ...)`: Client có thể gửi được **100** request.
> - **Tại sao:** Đoạn `min` đóng vai trò giới hạn dung lượng tối đa của "cái xô" (bucket). Nếu không có `min`, token sẽ liên tục được cộng dồn theo thời gian (10 phút x 10 token/phút = 100 token). Khi đó, client có thể xả toàn bộ 100 token cùng một lúc gây quá tải (burst) cho server, phá vỡ ý nghĩa của việc giới hạn capacity ban đầu.

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là bao nhiêu và service tự hồi phục khi nào?

> - **Hạn mức $30/tháng:** Thiệt hại tối đa là **$30**. Sự cố lúc 2h sáng có thể đốt sạch toàn bộ $30 chỉ trong vài giờ. Sau đó, service (hoặc client đó) sẽ bị chặn hoàn toàn và chỉ tự hồi phục vào **ngày đầu tiên của tháng tiếp theo**.
> - **Hạn mức $1/ngày:** Thiệt hại tối đa trong ngày hôm đó chỉ là **$1**. Đạt đến ngưỡng này, app sẽ chặn các request tiếp theo trong ngày, bảo vệ tài nguyên hệ thống. Service sẽ tự động hồi phục vào **0h00 ngày hôm sau**, giúp client vẫn sử dụng được dịch vụ vào các ngày tiếp theo trong tháng mà không bị "cắt đứt" cả tháng.

### Câu 9 — `/healthz` khác `/readyz` (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm 3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> 1. Redis mất kết nối.
> 2. Kubelet/Load Balancer gọi endpoint `/healthz` (đã bị gộp chung) và nhận được mã lỗi 500 do không ping được Redis.
> 3. Hệ thống quản lý lầm tưởng rằng cả 3 container đang bị chết/treo cứng (chứ không phải chỉ là chưa sẵn sàng phục vụ).
> 4. Hệ thống sẽ tiến hành "kill" và khởi động lại (restart) cả 3 container liên tục.
> 5. Hậu quả là toàn bộ dịch vụ bị downtime hoàn toàn. Lẽ ra, nếu tách riêng, khi Redis chết, app chỉ bị đánh dấu là "Unready" (rút khỏi Load Balancer, không nhận request) nhưng vẫn sống (Healthy) và tự động phục vụ lại khi Redis hồi phục.

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check timeout, sai `REDIS_URL`, app không đọc `$PORT`...): thông báo lỗi là gì, bạn tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> **Lỗi gặp phải:** Health check timeout / "Web process failed to bind to $PORT within 60 seconds of launch".
>
> **Tìm nguyên nhân:** Tôi xem trong mục Logs của Cloud Provider (ví dụ: Render/Railway) và thấy container khởi động thành công, nhưng cloud báo lỗi không tìm thấy cổng (port) đang lắng nghe. Kiểm tra code FastAPI, tôi nhận ra mình đã hard-code `port=8000` (như khi chạy trên localhost).
>
> **Cách sửa:** Tôi đã sửa lại mã nguồn để app tự động đọc biến môi trường `$PORT` do nhà cung cấp Cloud cấp phát động thay vì gắn cứng cổng 8000, bằng cách dùng: `port = int(os.getenv("PORT", 8000))` và cấu hình host thành `0.0.0.0`. Sau đó commit code lại và deploy thì thành công.
