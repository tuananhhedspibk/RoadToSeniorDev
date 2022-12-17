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

Tương ứng một-một giữa worker process và CPU là một sự lựa chọn cho backend.

## Connection Management

Worker processes lắng nghe ở cổng 80 cho HTTP. Việc lắng nghe ở cổng 80 này sẽ dẫn đến việc tạo ra:

- Một socket
- 2 queues

Trong đó có 1 queue `SYN` và 1 queue `ACCEPT`. Cả 2 queue này đều được quản lí bở kernel.

Khi client kết nối tới cổng 80, nó sẽ tạo 1 kết nối TCP với nginx server. Sau đó:

1. Client gửi `SYN` để bắt đầu quá trình TCP Handshake.
2. Kernel nhận `SYN` và match nó với socket của nginx sau đó đưa tín hiệu `SYN` vừa nhận được vào SYN queue.
3. Kernel đáp lại với tín hiệu `SYN-ACK` để hoàn tất quá trình TCP Handshake.
4. Cuối cùng thì Client sẽ trả lời lại `ACK` để hoàn tất quá trình "bắt tay ba bước"

Các worker processes sẽ cạnh tranh connection trên `ACCEPT queue` - Sau khi connection được hoàn tất, kernel sẽ di chuyển connection đến với `ACCEPT queue`.

Khi worker process chấp nhận kết nối, file descriptor được tạo và worker process sẽ có trách nhiệm đọc toàn bộ dữ liệu từ connection.

> Worker process có thể lấy về connection cho mình bằng những cách khác nhau. Có những lúc các worker processes sẽ lắng nghe trên cùng một cổng và kernel sẽ phân bổ kết nối tới chúng. Cũng có những lúc các worker processes có master worker process nhận các kết nối và gửi trực tiếp kết nối này cho worker process

Mỗi một worker process có thể xử lí nhiều connections cùng một lúc.

Dưới đây là hình vẽ tóm tắt lại một phần nội dung của người dịch:

![Client](https://user-images.githubusercontent.com/15076665/208221622-308ed362-f7fb-4524-b20b-c192bfe452ed.png)

## Request Processing

Xét trường hợp clients kết nối tới NGINX load balancer. 1 client kết nối tới `Worker 1` và `Worker 2`
*Chú thích: Hình minh hoạ là do người dịch tự vẽ lại*

![1NGINX](https://user-images.githubusercontent.com/15076665/208222289-62e891a6-63a6-4331-8851-16db0e8c2aca.png)

Client kết nối tới `Worker 1` sẽ gửi HTTP request để lấy về trang HTML. Đây là những gì sẽ xảy ra tiếp theo:

1. Request được chuyển thành `bytes stream` trên kết nối TCP.
2. Byte stream đi đến `buffer của kernel` để tiến hành kết nối.
3. Worker 1 sẽ tiến hành "nhận kết nối".
4. Dữ liệu liên quan tới kết nối sẽ được copy lại vào bộ nhớ của Worker 1.
5. Worker 1 sẽ tiến hành parse dữ liệu và hiểu rằng đây là một HTTP request.
6. Worker 1 sẽ xử lí request và đọc dữ liệu về trang HTML từ đĩa ra.
7. Worker 1 sẽ ghi HTTP response.
8. Response byte stream sẽ đi đến `send buffer`.
9. Kernel gửi dữ liệu về cho client.

Chú ý rằng, kernel sẽ KHÔNG BAO GIỜ push dữ liệu cho worker processes. Thay vào đó, nó sẽ giữ dữ liệu trong `receive buffer` và nói với worker process là trong kernel hiện có dữ liệu cần phải đọc.

> Tuỳ thuộc vào version của HTTP cũng như việc TLS có được sử dụng hay không, Worker process có thể sẽ phải đảm nhận nhiều công việc hơn khi tiến hành giải mã dữ liệu. Thao tác phân tích giao thức đối với HTTP/2 và 3 là tốn kém hơn so với HTTP/1.1. Việc có quá nhiều processes sẽ gây ra hiện tượng context switch (processes được swap in/ out khỏi CPU) và có thể làm ảnh hưởng đến hiệu năng.

Các bước kết nối đối với Worker 2 cũng hoàn toàn tương tự. Điểm khác biệt ở đây là Worker 2 sẽ khởi tạo một kết nối mới nhằm forward request đến đó. Đây cũng là lúc chúng ta cần để ý đến `backend connection pool`. Trên thực tế, đây là một lí do tại sao [Cloudflare](https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/) không sử dụng nginx nữa, `backend connection pool` không hề dễ dàng chia sẻ giữa các worker processes.

## Tổng kết

NGINX là một công cụ web server, load balancer và caching layer rất mạnh. Kiến trúc multiple worker cho phép NGINX có thể phân bổ các kết nối và request tới các CPUs khác nhau nhằm mục đích tối ưu hoá việc sử dụng tài nguyên. Đây chỉ là một phần tổng quan về kiến trúc bên trong của NGINX và vẫn còn rất nhiều điều để nói về NGINX.
