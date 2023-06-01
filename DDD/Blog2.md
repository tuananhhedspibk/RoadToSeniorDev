# Tôi đã xây dựng một API đơn giản bằng DDD như thế nào ? - Phần 2

Tiếp nối [phần 1](link) về DDD trong bài viết phần 2 lần này tôi xin phép được giới thiệu đến bạn đọc các kiến trúc thường dùng với DDD cũng như các khái niệm khác xoay quanh domain trong DDD, rất mong bạn đọc đón nhận nồng nhiệt.

## Các kiến trúc thường dùng với DDD

Khi áp dụng tư tưởng của DDD vào thiết kế hệ thống ta thường sẽ bắt gặp những kiểu kiến trúc phổ biến như sau:

- Kiến trúc 3 tầng (3 layers architecture).
- Kiến trúc phân tầng (Layerd architecture).
- Kiến trúc Hexagonal (Hexagonal architecture - Port and Adapter architecture).
- Kiến trúc "sạch" (Clean architecture).
- Kiến trúc "củ hành" (Onion architecture).

Chúng ta sẽ đi lần lượt các kiểu kiến trúc phía trên một cách tổng quan và tóm lược nhất có thể.

### Kiến trúc 3 tầng (3 layers architecture)

Phác thảo qua về hình dáng của nó:

![Screen Shot 2023-06-02 at 7 56 43](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/270a1d04-52ee-45c6-bb8c-1ef8156a0828)

Đây là một kiến trúc phổ biến được nhiều hệ thống web sử dụng, ngoài việc phổ biến thì việc triển khai nó cũng không quá khó thế nhưng nó lại có những nhược điểm lớn như sau:

- Các tầng sẽ có tính ràng buộc và phụ thuộc nhau rất lớn dẫn đến việc khó bảo trì hoặc làm mới từng tầng.
- Tính liên kết với một business logic class là rất thấp.

Để minh hoạ cho điều trên chúng ta cùng tìm hiểu ví dụ sau:

Giả sử ta có một hệ thống quản lí task với các chức năng cơ bản như sau:

- User không thể đăng kí nhiều mail cùng 1 lúc.
- User khi được tạo mới sẽ có trạng thái là `USING` nhưng cũng có thể chuyển đổi thành `SUSPENDING`.
- Task chỉ có thể được gán cho các user có trạng thái là `USING`.
- Task chỉ có 2 trạng thái là `COMPLETE` hoặc `DOING`.

Sơ đồ use-case của nó sẽ như sau:
