# System Design basic: bước đầu làm quen với Caching

Bài viết được dịch từ [nguồn](https://medium.com/towards-data-science/system-design-basics-getting-started-with-caching-c2c3e934064a)

## Cache

Cache là cơ chế giúp việc truy xuất dữ liệu trở nên nhanh hơn so với các data source. Cache thường được sử dụng để lưu các response hay được user request tới, ngoài ra cache còn được tận dụng để lưu kết quả của các xử lí tốn nhiều tài nguyên tính toán cũng như thời gian tính toán.

Cache sẽ lưu dữ liệu ở một nơi khác so với data source đang dùng với mục đích tăng tốc truy vấn. Cũng như load balancer, cache có thể được sử dụng ở nhiều vị trí khác nhau trong hệ thống.

Trong thiết kế hệ thống, khái niệm về cache thường được đề cập với mục đích:

- Cải thiện độ trễ của hệ thống.
- Giảm số lượng requests đến hệ thống.

Do cache thường được sử dụng để lưu các dữ liệu thường xuyên được sử dụng trong hệ thống, do đó nó giúp giảm lượng truy vấn đến DB.

## Các khái niệm trong cache

### Cache hit

Là khi truy vấn, ta tìm được dữ liệu cần thiết từ cache.

### Cache miss

Là khi truy vấn, ta không thấy dữ liệu từ cache.

### Data stale

Dữ liệu bị coi là `stale` nếu **dữ liệu trong primary database** được cập nhật mới nhất, trong khi **dữ liệu trong cache** thì không.
Thế nhưng bản thân việc dữ liệu bị stale không phải là một vấn để quá nghiêm trọng với hệ thống. Ta lấy ví dụ với dữ liệu số lượng viewer trên youtube, con số này không nhất thiết phải giống nhau với mọi người dùng, do đó ta có thể bỏ qua nó. Việc bỏ qua stale data problem sẽ giúp phát huy vai trò của cache.

