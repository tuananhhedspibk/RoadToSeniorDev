# Hiểu về Materialized View (Phần 1)

Bài viết được dịch từ [nguồn](https://medium.com/event-driven-utopia/understanding-materialized-views-bb18206f1782)

## Building Twitter clone

Thử tưởng tượng, bạn đang build một trang twitter clone, home (news feed) page sẽ là nơi chứa nhiều thao tác đọc nhất (ở đây đó là đọc các dữ liệu liên quan đến các bài đăng như nội dung, user picture, user name, ... của các users mà mình đang theo dõi).

Phía backend sẽ chạy câu query sau:

```SQL
SELECT tweets.*, users.*
FROM tweets
JOIN users
  ON tweets.user_id = users.id
JOIN follows
  ON follows.followee_id = users.id
WHERE follows.follower_id = $userId;
ORDER BY tweets.time DESC
LIMIT 100;
```

Khi chạy câu query trên, DB sẽ phải parse, tối ưu hoá, ... câu query sử dụng wall clock time, CPU time, ... ở đây kể cả khi ta chưa bàn đến việc JOIN các bảng lại với nhau thì việc chạy câu query với một lượng dữ liệu được chích xuất (có thể là khá lớn) như trên ở mỗi lần đọc là một thao tác vô cùng tốn kém.

Ở đây tôi đang nói về việc cả triệu users cùng chạy câu query như trên cùng một lúc trên DB của bạn, vậy rằng DB của bạn có thể chống đỡ được bao lâu và chống đỡ như thế nào ?

Và đây chính là lúc chúng ta vận dụng Materialized view.

## Materialized views

Materialized views sẽ tạo ra một pre-aggregated, read-optimized version cho source data nhằm mục đích giảm tải cho câu query mà ta cần chạy.

Ngay khi định nghĩa Materialized views xong, câu query sẽ được chạy và kết quả của nó sẽ được lên đĩa theo form của một table thông thường.

Khi tạo một materialized view, dữ liệu sẽ được truy xuất từ view thay vì thực thi lại câu query do bản thân câu query đã được thực thi trước đó. Điều này cực kì hữu ích, đặc biệt là khi câu việc chạy query tốn nhiều tài nguyên và chi phí.

## Làm cách nào để build một Materialized Views ?

Ta có thể định nghĩa một Materialized view thông qua câu `SELECT` như sau:

```SQL
SELECT MATERIALIZED VIEW timeline AS
SELECT user_id, display_name, avatar_url
FROM foo
JOIN bar ON ...
```

Materialized views là một read-optimized, denormalized structure. Do đó câu SELECT cấu thành nên view sẽ bao gồm:

- Phép `JOIN`
- Các hàm thống kê như `sum()`, `avg()`, `count()`

Khi tạo một materialized view, DB sẽ làm những công việc như sau:

- Scan toàn bộ các bảng liên quan để lấy ra consistent snapshot
- Chạy các hàm thống kê (aggregate)
- Chạy câu SELECT trên dữ liệu
- Copy kết quả thu được lên một bảng tạm nằm ở ổ đĩa

<img width="511" alt="Screenshot 2023-04-15 at 15 53 48" src="https://user-images.githubusercontent.com/15076665/232193696-40b850c8-2b57-4942-9df1-3c2ca7523e50.png">

Build materialized view là một quá trình tốn kém chi phí thế nhưng nó giúp tiến kiệm và đảm bảo về hiệu năng ở mỗi lần đọc dữ liệu.

## Materialized views vs tranditional views

Materialized view khác hoàn toàn so với "view" truyền thống.

View truyền thống (non-materialized view)　chỉ đơn thuần là alias của một câu query, khi đọc từ view, DB sẽ chuyển nó thành một câu query và chạy dưới nền của DB. Việc này giúp ta không phải bận tâm đến độ phức tạp của câu query nhưng nó lại không mang lại bất kì một hiệu quả gì về mặt hiệu năng cả.

Lấy ví dụ với câu lệnh tạo view như sau:

```SQL
CREATE VIEW vip_customers AS
SELECT customers.id, customers.name, SUM(orders.total) as total_spend
FROM customers
JOIN orders ON customers.id = orders.customer_id
GROUP BY orders.customer_id
HAVING SUM(orderss.total) > 5000;
```

Giả sử ta thực hiện `SELECT * FROM vip_customers`, query planner sẽ viết lại câu query vừa rồi dựa theo câu query gốc đã được aliased, do đó cứ mỗi khi ta tiến hành truy xuất vào view thì các phép join cũng như các hàm thống kê lại được thực thi. Ở đây không có bất kì dữ liệu nào được cached lại cả.

<img width="714" alt="Screenshot 2023-04-15 at 16 01 47" src="https://user-images.githubusercontent.com/15076665/232194073-a6f30cf0-e85c-4b6e-98a1-241ac85b7d28.png">

## Giữ cho view đồng bộ với source tables

Khi có dữ liệu mới được thêm vào bảng, làm cách nào để ta có thể đưa những dữ liệu mới này vào view. Về cơ bản ta có 2 cách đó là:

- **Full rebuild** materialized view.
- **Incremental refresh** materialized view.

### Full rebuild

Ở đây ta sẽ chạy lại câu query gắn với materialized view, do quá trình này rất tốn kém (nguyên nhân là bởi sẽ phải join lại các bảng và chạy lại các hàm thống kê) nên ta chỉ nên thực thi full rebuild materialized view theo từng khoảng thời gian nhất định để tránh ảnh hưởng đến hiệu năng của DB.

### Incremental refreshes

Ngược lại với full rebuild, phương pháp này sẽ tiến hành "merge" các delta data với materialized view hiện thời, phương pháp này nhanh và ít tốn kém hơn so với full rebuilds.

<img width="780" alt="Screenshot 2023-04-15 at 16 18 48" src="https://user-images.githubusercontent.com/15076665/232194885-57be4ee4-ab4b-4abf-a7f9-5e8858085479.png">

## Những vấn dề gặp phải khi sử dụng materialized views

Quay trở lại với ví dụ về twitter timeline, ta thấy rằng việc sử dụng materialized view cho user timeline data sẽ giúp ích rất nhiều về mặt hiệu năng của hệ thống. Khi ta chỉ cần lấy ra dữ liệu đã được lưu sẵn trong materialized view và trả về cho UI là xong.

Thế nhưng tron thực tế việc lưu dữ liệu liên quan đến view đòi hỏi phải có những chiến lược cụ thể như:

- Khi nào cập nhật dữ liệu cho view (nếu cập nhật bằng full rebuild sẽ cần phải cân nhắc tới hiệu năng của DB).
- Cần chọn refresh schedule phù hợp để có sự cân bằng giữa stale data và chi phí của quá trình rebuild.
