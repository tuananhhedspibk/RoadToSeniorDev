# Database

## RDS

RDS có 2 features chính:

- Multi-AZ: tránh thiên tai
- Read replica: cải thiện performance

Hình minh hoạ cho `Multi-AZ`

![File_000(1)](https://user-images.githubusercontent.com/15076665/198822396-0a2db7a7-6d0c-402b-b660-5ace838da383.png)

Hình minh hoạ cho `Read replica`

![File_000](https://user-images.githubusercontent.com/15076665/198822399-0629d316-9c28-432b-a9cb-a2bd9514a78a.png)

Read replica sẽ được đồng bộ hoá `asynchronous`

- Chạy trên virtual machine nên ta không thể login vào OS của RDS
- RDS `không phải serverless`

Bản thân read-replica cũng có thể promote để trở thành master. Read-replica chỉ có thể có khi RDB backup được bật.

### Backup

Có 2 loại backup:

- Automated backup: cho phép ta có thể backup dữ liệu ở bất kì thời điểm nào (trong vòng 0 -> 35 ngày: khoảng thời gian này được gọi là `retention period`). Quá trình này sẽ lưu snapshot của nguyên một ngày và `transaction logs` trong ngày đó. AWS sẽ chọn ra bản backup gần nhất, sau đó apply transaction logs từ lúc đó cho đến hiện tại (backup data được lưu trong S3)
- Database snapshot: thực hiện bằng tay, vẫn được lưu ngay cả khi original database bị xoá

Encryption đươch hỗ trợ và thực hiện bởi AWS KMS

Nếu RDS instance được mã hoá thì toàn bộ dữ liệu bên dưới cũng sẽ được mã hoá

## ElastiCache

In-memory cache in cloud.

Sử dụng để cải thiện hiệu năng khi truy xuất dữ liệu, có thể truy xuất dữ liệu từ in-memory cache thay vì disk-based database

Hỗ trợ 2 engine:

- redis: hỗ trỡ multi AZ
- memcached

## DynamoDB

NoSQL DB. Store on SSD storage.

**Eventual Consistent Read (Default):** Consistency với mọi bản copies dữ liệu trong vòng 1s

**Strongly consistent Read:** trả về kết quả phản ánh mọi thao tác ghi thành công

> one second rule: nếu cần đọc dữ liệu được cập nhật trong vòng <= 1s thì hãy sử dụng Strongly consistent, nếu không cần đọc dữ liệu cập nhật trong vòng 1s hãy sử dụng Eventual Consistent

### Key & Secondary index

![Screenshot 2023-03-30 at 9 33 43](https://user-images.githubusercontent.com/15076665/228698172-a942bef5-4480-41ad-90fc-43f53861bf3b.png)

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html

### DyanmoDB Accelerator (DAX)

- In-memory cache
- Dev không cần phải quản lí cache logic
- Giảm request time xuống cỡ `microsecond`

![File_000](https://user-images.githubusercontent.com/15076665/198825213-cbed1d5b-d410-492b-a860-436a454f6d8f.png)

### Transactions

- "All-or-nothing" operation
- read or write → prepare/commit

### On-demand capacity

- `Pay-per-request` pricing
- Balance cost & perfomance

### On-demand backup & restore

- Full backup tại mọi thời điểm
- Không ảnh hưởng đến hiểu năng hoặc availability
- Chỉ thực hiện được trong một region duy nhất

### Point-in-time recovery (PITR)

Tránh những thao tác ghi hoặc xoá nguy hiểm

### Stream

![Screen Shot 2022-10-29 at 19 02 02](https://user-images.githubusercontent.com/15076665/198825456-58d7c671-47af-48f9-8e91-7697c0b05a4b.png)

Mỗi một `stream record` sẽ tương đương với 1 database modification action. Các records này sẽ được tập hợp thành 1 shard

Có thể kết hợp thêm với `Lambda function` để thực thi như một `store procedure`

### Global table

- Based on DynamoDB streams
- Globally distributed application
- Quản lí `multi-master`, `multi-region replication`

### DMS

![Screen Shot 2022-10-29 at 19 07 33](https://user-images.githubusercontent.com/15076665/198825697-fde737b9-e8e1-48ae-9ecc-083a1dc4b7cc.png)

### Security

- Encryption sử dụng KMS
- Site-to-site VPN
- Direct connect (DX)
- Cloudwatch & Coudtrail

## Redshift

Data warehousing

Redshift có thể replicate asynchronous snapshot vào S3 ở region khác để tránh rủi ro do thảm hoạ, thiên tai

### Configured

- Single Node
- Multi-Node
  - Leader node: nhận queries
  - Compute node: thực thi queries (có thể lên tới 128 nodes)

Khi tiến hành nén, Redshift sẽ tự động chọn ra scheme tối ưu cho việc nén tuỳ theo dữ liệu đầu vào

### Encrypt

- Encrypt trong transit sử dụng SSL
- Sử dụng giải thuật AES-256

## Aurora

AWS Relational Database

Automated backup. Quá trình backup cũng như lấy snapshot không hề ảnh hưởng gì đến hiệu năng

Khi tạo ra, sẽ có 1 `aurora cluster` với bên trong là:

- 1 reader instance
- 1 writer instance

Nên trong 1 AZ sẽ có ít nhất 2 instances (2 bản sao lưu dữ liệu)

Trong số MySQL, PostgreSQL, Aurora thì chỉ có Aurora replica mới có chức năng `failover`

## Database Migration Service (DMS)

Là công cụ migration, di trú dữ liệu từ:

- RDB bên ngoài
- NoSQL bên ngoài
- Data warehouse bên ngoài

vào các Database service của AWS

![Screen Shot 2022-10-29 at 22 17 22](https://user-images.githubusercontent.com/15076665/198833656-6c466427-3ab7-4bb9-9355-f13d53a4b044.png)

DMS hỗ trợ:

- Migrate đồng nhất. VD: Oracle → AWS Oracle
- Migrate không đồng nhất. VD: SQL Server → AWS Aurora

Trong thực tế, migrate đồng nhất sẽ hoạt động như sau:

![Screen Shot 2022-10-29 at 22 21 41](https://user-images.githubusercontent.com/15076665/198833848-8203920b-8672-44e5-9fdd-1e9029cf942f.png)

Với migrate không đồng nhất thì ngoài DMS ta còn cần `Schema Conversion Tool - SCT`

![Screen Shot 2022-10-29 at 22 34 44](https://user-images.githubusercontent.com/15076665/198834496-6ff98ffd-ad54-4d32-b376-ec2bb612c5eb.png)

## Caching Strategies on AWS

Những services dưới đây có khả năng hỗ trợ caching:

- Cloudfront
- API Gateway
- ElastiCache: `Memcached` & `Redis`
- DynamoDB Accelerator (DAX)

Như hình phía dưới, ta có thể cache ở `cloudfront`, `API Gateway`, `ElastiCache` thậm chí cả DynamoDB (nếu ta sử dụng DAX).

![Screen Shot 2022-10-29 at 22 41 13](https://user-images.githubusercontent.com/15076665/198834833-1bb84f53-a443-4cff-873c-6f681f2e3831.png)

Và càng đi vào sâu trong hệ thống thì độ trễ sẽ tăng dần.

Caching cần đảm bảo cân bằng giữa:

- Up-to-date
- accurate information
- Latency

## EMR

Công cụ xử lí big-data. Trái tim của nó là cluster - container của các EC2 instances

Mỗi instance được coi như là 1 node với role cụ thể.

1 cluster sẽ có:

- Master node: quản lí cluster - bắt buộc có
- Core node: chạy task, lưu data vào `Handoop Distributed File System - HDFS`
- Task node: chỉ chạy task, không lưu data vào `HDFS`
