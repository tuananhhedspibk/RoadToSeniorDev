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
