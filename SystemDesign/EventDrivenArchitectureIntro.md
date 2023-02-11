# Giới thiệu về kiến trúc hướng sự kiện (Event-Driven Architecture)

Bài viết được lược dịch từ [nguồn](https://medium.com/microservicegeeks/introduction-to-event-driven-architecture-e94ef442d824)

## Định nghĩa đơn giản

Event-driven Architecture (EDA - Kiến trúc hướng sự kiện) là một kiến trúc phần mềm mà ở đó ta chú trọng vào việc tạo và sử dụng các **events** (sự kiện).

**Một event sẽ biểu diễn một hành động có mục đích nào đó**. Thông thường, các sự kiện có thể tương ứng với việc tạo hoặc thay đổi state của một vài entities. Ví dụ: khi ta mua một sản phẩm nào đó trên các trang e-commerce thì đó cũng là một event. Khách hàng gửi review về một sản phẩm nhận được đó cũng là một event.

## Event không bao giờ xảy ra

Một trong những đặc trưng của events đó là chúng không hề giao tiếp với các đối tượng sử dụng chúng. Event "chỉ xảy ra" mà thôi. Tuy vậy đó là là một yếu tố khiến event trở nên "rất mạnh" - thực tế là **event sẽ chuyển thành một record** bao hàm các thông tin liên quan đến sự kiện vừa xảy ra, cũng như các emiiters và quan trọng nhất đó là **nó tách biệt hoàn toàn đối với handlers**. Sự thật là **event producers** không hề biết đến sự tồn tại của **event consumers**.

**Một record chỉ bao hàm các thông tin liên quan đến sự kiện vừa xảy ra mà thôi**. Như ở ví dụ kể trên về việc mua đồ trên trang e-commerce, event mua đồ có thể được mô tả bằng JSON object như sau:

```JSON
{
  "orderId": "760b5301-295f-4fec-95f8-6b303a3b824a",
  "customerId": 28623823,
  "productId": 31334,
  "quantity": 1,
  "timestamp": "2021-02-09T11:12:17+0000"
}
```

> Chú ý rằng: trong thực tế records và events là hai thuật ngữ có thể dùng thay thế cho nhau. Ví dụ: thuật ngữ "event" được sử dụng để nói về "record" của sự kiện đó.

Ứng dụng của chúng ta tạo ra một event, tuy nhiên nó không hề biết *Ai* sẽ xử lí event đó, *khi nào*, *Ở đâu* và *Xử lí như thế nào*.

Bản thân **event producer** chỉ đảm bảo cung cấp đủ thông tin để **event consumer** có thể xử lí event mà thôi.

Do đó các thông tin khác, ví dụ như thông tin về `stock count` của sản phẩm hoặc `địa chỉ gửi đến` của sản phẩm sẽ không được **event producer** quan tâm ở đây.

## Channelling Events

Nếu **event producers** và **event consumer** không hề biết gì về nhau, vậy thì chúng sẽ giao tiếp với nhau bằng cách nào ?

Events thường được lưu trữ ở một nơi được gọi là **log** (Đôi khi thuật ngữ **ledger** có thể được sử dụng). Logs là một dạng cấu trúc dữ liệu append-only bậc thấp, log là nơi mà producer sẽ lưu các events để sau đó các consumer có thể truy cập và lấy về các events. Mọi thao tác với log đều do **brokers** - một middleware nằm ở giữa producers và consumers đảm nhận. Khi một event được publish thì tất cả mọi đối tượng đều có thể sử dụng event đó.

Khi triển khai event-driven systems, chúng ta thường sử dụng thuật ngữ **stream** để chỉ một hoặc nhiều logs. Log là một thuật ngữ mang tính chất "vật lý" khi nó sử dụng các file để lưu trữ, còn stream mang tính logic khi nó là một **chuỗi các events**. `Apache Kafka` là một streaming platform khá bổ biến - các streams được để cập đến bởi 2 thuật ngữ **topics** & **partitions**.

Mối liên hệ giữa producers, consumers và streams có thể được mô tả như mô hình dưới đây

![Screen Shot 2023-02-11 at 23 04 05](https://user-images.githubusercontent.com/15076665/218262320-929052fb-b174-4dd2-91b9-3a39b285e953.png)
