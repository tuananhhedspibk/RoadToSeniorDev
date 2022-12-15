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
