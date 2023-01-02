# Thiết kế hệ thống như Instagram

## Requirement

1. User có khả năng upload ảnh từ thiết bị di động.
2. Users có khả năng "follow" các users khác.
3. Danh sách các bài đăng (post).
4. Scale: 10 triệu user.

## Scale

- Giả sử hệ thống có 10 triệu users active mỗi tháng.
- 1 user upload trung bình 2 bức ảnh mỗi tháng.
- 1 bức ảnh có dung lượng 5MB.

=> Ta cần: 10 triệu *2* 5MB = 100,000,000 MB lưu trữ = 100TB ~ 1.2PB

## Data types

[Screen Shot 2023-01-02 at 19 14 39](https://user-images.githubusercontent.com/15076665/210219618-bbaf64ea-6d08-4ca4-92d4-a05ee87200ee.png)

## High level design

[Screen Shot 2023-01-02 at 19 29 23](https://user-images.githubusercontent.com/15076665/210219623-9b0797b0-1b9f-4503-b24e-c8612571cb28.png)
