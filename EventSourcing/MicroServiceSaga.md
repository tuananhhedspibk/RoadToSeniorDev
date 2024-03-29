# Saga Pattern trong Microservice

Bài viết được dịch từ nguồn: <https://microservices.io/patterns/data/saga.html>

## Ngữ cảnh

Nếu bạn đã từng triển khai **Database per Service** pattern (mỗi service sẽ có cho riêng mình một DB) bạn có thể thấy rằng, theo yêu cầu của business logic ta cần triển khai transaction trên nhiều services khác nhau, ví dụ như với một trang EC, mỗi người dùng (ở đây là khách hàng - custimer) sẽ có hạn mức (credit limit) của riêng mình, hệ thống cần đảm bảo rằng order mới này không vượt quá hạn mức của khách hàng, nhưng do Order Và Customer là 2 DB khác nhau, trực thuộc 2 services khác nhau nên do đó hệ thống không thể triển khai ACID transaction như với 1 DB thông thường được.

## Vấn đề

Vậy vấn đề ở đây đó chính là **làm cách nào để triển khai transaction trên nhiều services ?**

## Giải pháp

Ta sẽ sử dụng Saga cho trường hợp này, nói một cách đơn giản thì saga là một chuỗi các local trasactions.

> Mỗi một local transaction sẽ update DB đồng thời publish message hoặc event để trigger local transaction của service kế tiếp trong chuỗi logic nghiệp vụ

Trong trường hợp một transaction bị failed do vi phạm nghiệp vụ, ta sẽ phải tiến hành rollback toàn bộ các transaction đã "thành công" trước đó.

![Screen Shot 2023-10-26 at 22 21 51](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/a1f84f04-9692-4e09-bb52-a34221b8a669)

Có hai cách triển khai saga:

1. Choreography: mỗi local transaction sẽ publish domain event, event này sẽ trigger local transactions ở các services khác (các services khác sẽ tự mình phán đoán xem có cần phải xử lí hay không).
2. Orchestration: mỗi orchestrator (object) sẽ chỉ định cụ thể các local transactions sẽ được thực thi.

## Choreography-based saga

Follow triển khai order trên trang EC theo như choreography sẽ như sau:

![Screen Shot 2023-10-26 at 22 42 30](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/f9cdc07e-1fed-4e58-b1ef-4146cd6680b8)

1. `Order Service` nhận `POST /orders` request, tạo `order` với trạng thái là `PENDING`.
2. Sau đó nó sẽ emit một `Order Created` event.
3. `Customer Service` lắng nghe sự kiện này và trigger `reserve credit handler`.
4. Sau đó `Customer Service` cũng sẽ emit một `Credit Reserved` event.
5. `Order Service` lắng nghe sự kiện `Credit Reserved` và cập nhật trạng thái của `Order` thành `COMPLETE` hoặc `REJECT`

## Orchestration-based saga

![Screen Shot 2023-10-27 at 7 44 42](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/0a36b90f-3124-4a32-9828-cda710ce6502)

Giải thích sơ qua về Orchestration-based như sau:

1. `Order Service` nhận `POST /orders` request và tạo ra `Create Order` saga orchestration.
2. Saga orchestrator tạo ra `Order` với trạng thái `PENDING`.
3. Sau đó sẽ gửi `Reserve Credit` command sang `Customer Service`.
4. `Customer Service` sẽ tiến hành chiếm hữu hạn mức order của user (reserve credit).
5. Sau đó nó sẽ trả ra message và chỉ đích danh saga nào sẽ được thực thi ở phía Order Service.
6. Saga orchestrator sẽ approve hoặc reject `Order`.

## Đánh giá chung

### Ưu điểm

Cho phép ứng dụng có thể đảm bảo tính thống nhất về mặt dữ liệu trên nhiều services khác nhau mà không cần đến distribution transaction.

### Nhược điểm

Programming model sẽ rất phức tạp.

### Những vấn đề có thể gặp phải

- Để có thể được tin tưởng thì mỗi service ngoài việc cập nhật DB của mình thì nó cũng cần phải publish message/event đến message broker thay vì sử dụng distribution transaction truyền thống.
- Client khởi tạo saga theo async flow nhưng lại sử dụng sync request (VD `POST /orders`) nên ta cần định nghĩa rõ ràng đầu ra, sẽ có một vài lựa chọn đi kèm với trade-offs khác nhau như sau:

  - Service sẽ trả về response sau khi saga kết thúc hoàn toàn (VD: khi `OrderAccepted` hoặc `OrderRejected` event được nhận)
  - Service trả về response (bao gồm `orderID`) ngay sau khi saga được khởi tạo, client sẽ định kì poll (VD: gọi tới `GET /orders/{orderID}`) để lấy về kết quả.
  - Service gửi về response (bao gồm `orderID`) sau khi saga được khởi tạo và gửi event (thông qua websocket, webhook) tới client khi saga kết thúc hoàn toàn.
