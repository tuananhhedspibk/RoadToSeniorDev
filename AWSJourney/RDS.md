# AWS — Difference between Multi-AZ and Read Replicas in Amazon RDS

Multi-AZ & Read replica cùng maintain một bản sao của DB thế nhưng về bản chất chúng có những sự khác biệt nhất định.

## RDS Multi-AZ

![Screenshot 2023-04-11 at 10 42 56](https://user-images.githubusercontent.com/15076665/231033698-dca2121c-13e6-4a78-96e1-35c8c03fb7d7.png)

Multi-AZ sử dụng 2 DB instance ở 2 zones khác nhau (cùng một region), ta có một Main DB và một Standby DB. Dữ liệu sẽ được đồng bộ thường xuyên giữa main và standby DB. Nếu có lỗi xảy ra ở main DB, lỗi sẽ được chuyển tiếp đến cho standby instance nhằm mục đích tránh ảnh hưởng đến app.

### Lợi ích của Multi-AZ

- Giảm ảnh hưởng của quá trìh maintaince. Khi tiến hành maintaince, đầu tiên standby instance sẽ được maintain, sau đó nó sẽ được đưa lên trở thành main instance, main instance khi đó sẽ trở thành standby và được tiến hành maintain.

### Multi-AZ usecase

- High availability trong quá trình vận hành cũng như khi maintain.
- Bảo vệ DB khỏi instance failure hoặc các sự cố với AZ.
- Giảm I/O freeze, giảm đi độ trễ khi hệ thống đang backup.

## RDS Read Replica

![Screenshot 2023-04-11 at 10 59 14](https://user-images.githubusercontent.com/15076665/231036201-206efbf1-db22-4579-aa03-25e1e2100591.png)

RDS Read replica cho phép ta tạo một hoặc nhiều bản sao DB instance trong:

- Cùng AZ
- Khác AZ (cùng region)
- Khác Region

Dữ liệu sẽ được cập nhật từ Master DB sang Replica DB một cách bất đồng bộ. Thao tác ghi sẽ nằm ở Master DB, thao tác đọc sẽ được chuyển hướng sang Replica DB.

Khi tạo replica, snapshot sẽ được lấy từ Master DB, sau đó đưa sang để tạo Read-Only Replica.

### Lợi ích của Replica

- Giảm tải cho Master DB (ở khâu đọc)
- Giúp maintaince DB ở các regions khác nhau nhằm tránh các vấn đề về thiên tai, thảm hoạ.

### Read Replica Usecase

- Business Reporting hoặc Data Warehousing sẽ là lúc cần dùng đển Read-Replica, do các câu queries thuộc nhóm này thường khá nặng, tốt hơn hết là không nên dồn chúng vào thực thi ở Master DB.
- Phục vụ cho các ứng dụng đọc nhiều.
- Đảm bảo quá trình đọc luôn được diễn ra thông suốt ngay cả khi Master DB ở chế độ Maintaince hoặc I/O suspension for backing up.

> Chú ý: Mục đích chính của Multi-AZ không phải là đề read scaling, bạn không thể sử dụng Standby Instance cho mục đích phụ vụ read traffic được, mục đích chính của nó là để chịu lỗi (failover). 

## Tài liệu tham khảo

- https://medium.com/awesome-cloud/aws-difference-between-multi-az-and-read-replicas-in-amazon-rds-60fe848ef53a
