# Redis Architecture

Tham khảo từ: https://architecturenotes.co/redis/

## Redis là gì ?

Redis là một `in-memory database`, lưu trữ dữ liệu dưới dạng `key - value`.

Redis thường được "đặt phía trước" main database (MySQL, PostgreSQL) để cải thiện hiệu năng hệ thống.

Dữ liệu thường được lưu trong redis là:

- Dữ liệu ít thay đổi nhưng được truy xuất thường xuyên
- Dữ liệu ít quan trọng và vẫn trong quá trình phát triển.

![Redis](https://user-images.githubusercontent.com/15076665/185725532-9e03dc0e-df48-4b1c-aeb8-a429bc5add93.png)

## Redis Architectures

### Single Redis Instance

Đơn thuần là chỉ sử dụng duy nhất 1 Redis instance hoặc cluster. Cách làm này rất dễ triển khai xong nhược điểm dễ thấy nhất đó là nếu instance duy nhất này bị "sập" thì client sẽ không thể truy cập vào cache dẫn tới hiệu năng của hệ thống bị ảnh hưởng.

Các lệnh gửi đến redis sẽ được xử lí ở trong memory. Nếu setup persistence cho redis thì sẽ có các tiến trình con chạy trong những khoảng thời gian cố định để lưu dữ liệu của redis vào RDB snapshot hoặc AOF (append-only files)

Nếu không lưu dữ liệu vào snapshot hoặc AOF thì khi redis "sập" toàn bộ dữ liệu sẽ mất sạch, ngược lại nếu lưu thì khi redis restarts thì dữ liệu sẽ được phục hồi trở lại.

![Single Redis Instance](https://user-images.githubusercontent.com/15076665/185725978-91fa0b26-b61d-4385-af00-687555aef325.png)

### Redis HA (High availability)

![Redis HA](https://user-images.githubusercontent.com/15076665/185726005-766bbc2c-2e54-498e-85a7-04d2487d0973.png)

Triển khai nhiều redis instances hơn và đồng thời tiến hành đồng bộ hoá dữ liệu giữa main instance với các instances khác.

Điều này đảm bảo hệ thống tránh được tình trạng "có một điểm chịu lỗi duy nhất". Từ đó:

- Tăng tính sẵn có của hệ thống
- Biết được hệ thống fail ở điểm nào để có biện pháp phục hồi thích hợp

### Redis Replication

Mọi main instance redis đều có 2 thuộc tính là `offset` và `Replication ID`.

Hai thuộc tính này giúp ta tìm ra thời điểm bắt đầu sync data là từ lúc nào. Offset sẽ tự động tăng lên sau mỗi action diễn ra trên main redis instance

Cụ thể như sau: khi một redis instance có offset nhỏ hơn so với offset của main instance thì quá trình sync data sẽ diễn ra cho đến khi 2 offset bằng nhau thì thôi

![Redis offset](https://user-images.githubusercontent.com/15076665/185727257-05b0261c-2e70-4bd9-8963-9b3917ec76cf.png)

Nếu ở 2 instances ta đều không biết offset thì sẽ diễn ra quá trình `full synchronization`. Primary instance sẽ tạo ra RDB snapshot và gửi đến cho replica.

Trong quá trình gửi snapshot, main instance sẽ buffer mọi `update commands` tính từ thời điểm tạo snapshot cho đến offset hiện thời của nó.

Vai trò của `replication ID` ở đây đó là giúp triển khai `partial synchroinization` thay vì `fully synchronization`.

Cụ thể như sau, khi một instance trở thành primary instance mới thì `relication ID` mới sẽ được sinh ra, đây có thể coi như ID của `ancestor instance`. Khi 2 instance có chung một ancestor thì quá trình sync data chỉ diễn ra với những phần chênh lệch được xác định dựa trên `offset`

![Partial Sync](https://user-images.githubusercontent.com/15076665/185728994-ca1e06ac-c0a0-413e-b580-3461c7720c0f.png)

Trong trường hợp `replication ID` khác nhau thì sẽ phải sync toàn bộ data giữa 2 instance.

![Fully sync](https://user-images.githubusercontent.com/15076665/185729013-717c7b49-2b35-4871-bf74-4657c56e231f.png)

### Redis Cluster

![Redis cluster](https://user-images.githubusercontent.com/15076665/185729554-b0c58382-9b21-4752-b498-a98e852665f7.jpeg)

Việc chỉ sử dụng duy nhất một instance ngoài nguy cơ single point failover còn có thể dẫn đến tình trạng instance có thể bị quá tải về mặc dung lượng do tài nguyên của server luôn là có hạn.

Giải pháp ở đây đó là áp dụng `horizontal scaling` cho cache layer như hình minh hoạ ở trên. Tuy nhiên vấn đề đặt ra ở đây đó là "cách thức" lưu trữ dữ liệu trên các shards (ở đây mỗi instance trong cluster được coi là một shard).

Việc lưu trữ dữ liệu trên nhiều máy được gọi là `sharding`. Để tìm nơi lưu trữ cho key ta sẽ thực hiện các bước như sau:

1. Sử dụng một `hash function` cho trước để băm key
2. Modulo kết quả dựa trên tổng số shard: `mod(number_of_shards)`

![Hash key](https://user-images.githubusercontent.com/15076665/185729943-d0efb502-b651-4c3f-9368-ef5a7314a80e.png)

Với công thưc trên thì key sẽ luôn được lưu trữ ở một shard duy nhất.