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
