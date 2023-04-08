# CQRS Design Pattern trong kiến trúc Microservices

Bài viết được dịch từ [nguồn](https://medium.com/design-microservices-architecture-with-patterns/cqrs-design-pattern-in-microservices-architectures-5d41e359768c)

Trong bài viết lần này, chúng ta sẽ nói về một **Design Pattern** của Microservices architecture đó là **CQRS Design Pattern**. Chúng ta sẽ sử dụng pattern này để thiết kế ứng dụng EC với kiến trúc Microservice.

![Screen Shot 2023-04-08 at 16 47 38](https://user-images.githubusercontent.com/15076665/230710205-a1949159-6213-4e9d-a023-0e2a682e21b8.png)

Hình phía trên chính là minh hoạ cho **CQRS Design Pattern**

Sau khi đọc xong bài viết này, hi vọng bạn đọc có thể rút ra cho mình những kiến thức cần thiết để áp dụng **CQRS Design Pattern** vào kiến trúc Microservice.

## CQRS Design Pattern

CQRS là một trong những pattern rất quan trọng khi tiến hành truy xuất/ querying giữa các microservices. CQRS có thể giúp cho chúng ta tránh được những queries phức tạp, CQRS là viết tắt của **Command and Query Responsibility Segregation**. Nói một cách đơn giản nó chia tách các thao tác **đọc** và **cập nhật** cho DB.

Với các ứng dụng monolithic, hầu hết ta chỉ có duy nhất 1 DB, DB này vừa đảm nhận nhiệm vụ nhận các câu truy vấn phức tạp, vừa đảm nhận thêm nhiệm vụ CRUD. Thế nhưng khi ứng dụng trở nên phức tạp hơn thì kéo theo đó các câu queries cũng như CRUD operations cũng sẽ trở nên phức tạp lên theo, lúc đó sẽ rất khó để kiểm soát hành vi của hệ thống.

Lấy ví dụ với việc đọc dữ liệu, nếu câu query của chúng ta phức tạp đến mức cần phải **join 10 bảng lại với nhau**, thì thời gian lock DB lại sẽ lâu hơn và từ đó độ trễ trong tính toán của câu query cũng sẽ tăng lên theo. Cũng tương tự như với thao tác ghi lên DB, khi thực thi các CRUD operations ta cần các validations phức tạp khiến cho thời gian xử lí tăng theo, dẫn đến **thời gian lock DB cũng theo đó mà tăng lên**.

![Screen Shot 2023-04-08 at 17 02 55](https://user-images.githubusercontent.com/15076665/230710750-3c232c7e-96b0-4f9b-8044-1c88fb56c569.png)

Với CQRS ta sẽ sử dụng 2 DB khác nhau cho reading và writing với các "chiến lược" khác nhau dành cho chúng.

Hơn nữa, ta cũng cần cân nhắc yêu cầu của hệ thống để đưa ra cách thiết kế phù hợp, với các ứng dụng "chuyên đọc" - **read-incentive application** ta nên thiết kế kiến trúc hệ thống sao cho nó tập trung cao độ vào reading DB.

**Command** trong CQRS sẽ tập trung vào việc cập nhật dữ liệu (VD: thêm item vào cart, checkout order, ...), do đó commands nên được xử lí với message broker systems cung cấp cách thức xử lí commands một cách bất đồng bộ.

**Queries** thì không chỉnh sửa gì dữ liệu cả, nó chỉ trả về JSON data với DTO objects.

Để có thể tách bạch hoàn toàn **Commands** và **Queries** ta sẽ tiến hành chia DB thành 2 DB phân biệt nhau. Bằng cách này, nếu ứng dụng của chúng ta là một ứng dụng "chuyên đọc", ta sẽ định nghĩa các custom schema để tối ưu hoá các câu queries.

**Materialized view pattern** là một ví dụ tốt cho việc triển khai reading DB, bằng cách này chúng ta có thể tránh được các phép JOINS phức tạp và có thể mapping với các schema được định nghĩa trước chuyên dùng cho thao tác query.

> Với cách làm như vậy ta hoàn toàn có thể sử dụng **NoSQL document database** cho reading và sử dụng RDB cho writing

## Instagram Database Architecture

Instagram sử dụng 2 DB systems:

- RDB **PostgreSQL**
- NoSQL **Cassandra**

![Screen Shot 2023-04-08 at 17 49 13](https://user-images.githubusercontent.com/15076665/230712704-f99e7301-293b-43cc-94de-023c8cd60556.png)

Điều đó có nghĩa rằng Instagram sử dụng **Cassandra** cho các **read-incentive data**. Sử dụng  **PostgreSQL** cho bio update.

## Làm cách nào để đồng bộ các DBs với CQRS ?

Khi tiến hành chia tách thành 2 DBs thì có một vấn đề mà ta cần lưu tâm đó chính là việc đồng bộ hoá dữ liệu giữa chúng.

Việc này có thể được triển khai bằng cách ứng dụng **Event-Driven Architecture**. Với **Event-Driven Architecture**, khi update trong main DB, ta sẽ tiến hành publish ra một **update event** sử dụng **message broker systems**, event này sẽ được **tiêu hoá** bởi read DB, sau đó quá trình **sync data** sẽ được tiến hành.

Tuy nhiên, một vấn đề khác sẽ phát sinh ở đây, đó là **consistency issue**. Vì chúng ta triển khai **async communication** với message broker, nên dữ liệu sẽ không được phản ánh ngay lập tức.

Từ đó ta đưa ra nguyên tắc **eventual consistency**. Read DB sẽ "đều đặn" đồng bộ hoá dữ liệu từ **write database**, nó sẽ mất một khoảng thời gian nhất định để cập nhật read DB một cách "bất đồng bộ". Chúng ta sẽ cùng nhau thảo luận về eventual consistency ở các phần tiếp theo.

Khi bắt đầu thiết kế, bạn có thể sử dụng read database bằng cách replicate từ write database. Bằng cách này chúng ta có thể sử dụng các **read-only replicas** khác nhau bằng việc áp dụng **Materialized view pattern** để từ đó tăng hiệu năng của câu query.

Hơn nữa, do ta tiến hành chia tách thành 2 DBs nên chúng hoàn toàn có thể được scale một cách độc lập với nhau tuỳ theo yêu cầu của hệ thống.

Việc sử dụng **Event-driven architecture** để đồng bộ hoá dữ liệu đòi hỏi chúng ta cần phải biết đến **Event sourcing pattern**.

## Thiết kế kiến trúc - CQRS, Event sourcing, Eventual Consistency, Maerilized View

Tôi tiến hành thiết kế kiến trúc cho ứng dụng EC của mình như sau:

![Screen Shot 2023-04-08 at 18 29 51](https://user-images.githubusercontent.com/15076665/230714276-052c5726-21b3-4c56-bbd1-8334e67cb766.png)

Với Ordering service, ta sẽ chia thành 2 nhóm DB:

- Nhóm 1: DB cho **writing** với bản chất là RDB.
- Nhóm 2: DB cho **reading** dùng cho querying.

Ta sẽ tiến hành đồng bộ hoá dữ liệu giữa 2 nhóm DB này bằng **message broker system** thông qua **publish/ subscribe pattern**

Và đây chính là tech stack tôi sẽ sử dụng cho ứng dụng EC của mình:

- **SQL Server** cho RDB (writing DB)
- **Cassandra** cho NoSQL DB (reading DB)
- **Kafka** cho việc đồng bộ dữ liệu 2 DBs

### Lời người dịch

Trên đây là bài dịch của tôi, hi vọng rằng bạn đọc có thể tìm được những thông tin hữu ích về CQRS cũng như Event sourcing, hẹn gặp lại bạn đọc ở các bài viết tiếp theo. Xin cảm ơn.
