# Kiến trúc của nginx

Chú thích: bài viết được dịch từ [nguồn](https://medium.com/@hnasr/the-architecture-of-nginx-2b32fc0b7877)

Nginx là một phần mềm mã nguồn mở, dùng để scale web server và reverse proxy. Nginx được sử dụng phổ biến như một công cụ giúp bảo vệ phần backend infrastructure.

Nginx đảm nhiệm các vai trò tại `Caching Layer`, `API Gateway`, `Web Server`.

Trong bài viết lần này tôi sẽ đi sâu vào kiến trúc bên trong của nginx trên các phương diện:

- Nginx worker processes
- Connection Management
- Request Processing

## Worker processes

Phần chính yếu trong kiến trúc của Nginx đó là `master process`. Khi khởi động nginx, nginx sẽ tự động khởi tạo một process (tiến trình) quản lí các tiến trình còn lại của nginx.

Master process sẽ tạo 2 processes cho `cache management`.

- Một process sẽ đọc cache từ ổ đĩa.
- Một process cho mục đích làm mới (refresh) lại cache.

Khi khởi động nginx ở chế độ tự động (đây là chế độ mặc định của nginx), nginx cũng sẽ khởi tạo một số lượng các worker processes nhất định dựa theo số lượng CPU cores mà bạn có. Số lượng processes này có thể được thay đổi tuỳ vào việc bạn có sử dụng `Hyper-threading` hay không

> Hyper-Threading cho phép một CPU core vật lý có thể xuất hiện như hai core logic đối với hệ điều hành. Điều này có thể giúp tăng hiệu năng của hệ thống vì hệ điều hành khi đó có thể lập lịch cho nhiều tiến trình hơn trên các CPU thread

> Nếu bạn có 2 CPU cores và sử dụng Hyper-Threading thì hệ điều hành có thể sử dụng 4 logic core hoặc thread. Bạn có thể cân nhắc sự lựa chọn này tuỳ theo hiệu năng mà mình mong muốn

Có một tương ứng một-một giữa worker process và CPU là một sự lựa chọn cho backend.