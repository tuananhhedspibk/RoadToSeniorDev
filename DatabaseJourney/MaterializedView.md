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
