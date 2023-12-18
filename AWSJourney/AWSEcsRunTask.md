# Sử dụng ECS Run Task cho mục đích tương tác giữa các micro-services

## Tương tác giữa các micro-services

Trong hệ thống gồm nhiều services, việc tương tác giữa các services là điều thường xuyên diễn ra. Các tương tác này có thể thông qua:

- API calling: đây là cách gọi API sử dụng HTTP hoặc RPC
- Event trigger: có thể hiểu là việc service A sẽ "gọi" đến service B thông qua việc emit một hoặc một vài events nào đó.

Về cách sử dụng API calling - đây là cách làm khá phổ biến nên tôi sẽ không đề cập đến trong bài viết lần này, thay vào đó tôi muốn đi sâu vào cách thứ hai, đó là sử dụng "Event trigger".

## SAGA pattern

Do mỗi một service sẽ có cho mình một DB riêng, nên việc service A "gọi" service B dẫn đến việc ta phải thực thi các transactions trên các DB khác nhau (chú ý rằng ta KHÔNG THỂ triển khai ACID transaction trên nhiều DB khác nhau).

Dẫn đến cần có một giải pháp để đảm bảo sử "thống nhất" giữa "các DB" với nhau.

Giải pháp ở đây đó chính là SAGA, có thể hiểu đơn giản rằng SAGA là một chuỗi các local transaction (transaction thuộc về một DB).

> Mỗi một local transaction sẽ update DB mà nó thuộc về, đống thời publish message hoặc event để trigger local transaction của service kế tiếp trong chuỗi logic nghiệp vụ

Như hình minh hoạ dưới đây.

![Screen Shot 2023-12-11 at 23 33 51](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/1385f989-20dc-4afb-b732-c121b8b6bf72)

Có 2 cách triển khai SAGA:

1. **Choreography**: mỗi local transaction sẽ publish _domain-event_, _domain-event_ này sẽ trigger local transactions ở các services khác (bản thân các services này sẽ tự mình phán đoán xem có cần phải xử lí hay không).
2. **Orchestration**: mỗi orchestrator (object) sẽ chỉ định cụ thể các local transactions sẽ được thực thi.

### Choreography-based SAGA

Tôi lấy ví dụ với một trang EC với 2 domains chính là **Order** và **Customer** lần lượt là:

1. Nghiệp vụ diễn ra mỗi khi người dùng mua hàng.
2. Nghiệp vụ liên quan đến khách hàng (người dùng).

Follow triển khai order trên trang EC theo như choreography sẽ như sau:

![Screen Shot 2023-10-26 at 22 42 30](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/f9cdc07e-1fed-4e58-b1ef-4146cd6680b8)

1. `Order Service` nhận `POST /orders` request, tạo `order` với trạng thái là `PENDING`.
2. Sau đó nó sẽ emit một `Order Created` event.
3. `Customer Service` lắng nghe sự kiện này và trigger `reserve credit handler`.
4. Sau đó `Customer Service` cũng sẽ emit một `Credit Reserved` event.
5. `Order Service` lắng nghe sự kiện `Credit Reserved` và cập nhật trạng thái của `Order` thành `COMPLETE` hoặc `REJECT`

### Orchestration-based SAGA

![Screen Shot 2023-10-27 at 7 44 42](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/0a36b90f-3124-4a32-9828-cda710ce6502)

Giải thích sơ qua về Orchestration-based như sau:

1. `Order Service` nhận `POST /orders` request và tạo ra `Create Order` saga orchestration.
2. Saga orchestrator tạo ra `Order` với trạng thái `PENDING`.
3. Sau đó sẽ gửi `Reserve Credit` command sang `Customer Service`.
4. `Customer Service` sẽ tiến hành chiếm hữu hạn mức order của user (reserve credit).
5. Sau đó nó sẽ trả ra message và chỉ đích danh saga nào sẽ được thực thi ở phía Order Service.
6. Saga orchestrator sẽ approve hoặc reject `Order`.

### Trigger event

Qua 2 phần định nghĩa ở trên, bạn có thể thấy rằng, dù là cách thức triển khai SAGA thế nào đi nữa thì một yếu tố luôn được cất nhắc ở đây đó chính là "Event".

"Event" được **EMIT** sẽ là "nguồn cơn" cho mọi thứ:

- Việc thực thi một nghiệp vụ ở một service khác (một cách "cưỡng ép")
- Việc "bắn tín hiệu" để service khác "tự mình" thực thi một nghiệp vụ cụ thể.

Cụ thể hơn về SAGA pattern tôi sẽ trình bày ở một bài viết khác.

## Thiết kế cách thức tương tác

Quay trở lại chủ đề chính đó là việc tôi đã triển khai cách thức tương tác thông qua ECS Run Task như thế nào ?

Nói nhanh thì ECS là:

> Một orchestration service cho phép deploy, manage và scale các containerized app

Vòng đời của một ECS application

Trong đó:

![Screen Shot 2023-11-18 at 22 42 22](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/70305f62-d7d1-4eb2-9ee9-cddcdab3b854)

- ECR sẽ lưu DockerImage tương ứng với app.
- Task Definition sẽ là blueprint của app (JSON file với các params, containers cấu thành nên app)

Với ECS ngoài việc dựng một server hoặc một ứng dụng thì ta cũng có thể chạy một "tác vụ" nào đó.

"Tác vụ" mà tôi nói đến ở đây chính là "Task", nó có thể là một nghiệp vụ (usecase) hoặc một con batch chẳng hạn.

Nó khác với server ở chỗ, server sẽ "duy trì" tình trạng chạy "mãi mãi" nghĩa là nó chỉ bị tắt đi khi có tác động từ phía developer.

Còn "Task" sẽ "tự động" tắt đi sau khi nó hoàn thành "nhiệm vụ" của mình, việc này có thể thấy ngay rằng sẽ giúp chúng ta giảm đi đáng kể chi phí vận hành thay vì lúc nào cũng "thường trực" chạy một con batch hoặc một nghiệp vụ nào đó.

### Kiến trúc sử dụng

Ở đây tôi dựng nên những thành phần chính như hình bên dưới:
