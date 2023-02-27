# Event-Driven System: Tìm hiểu về kiến trúc Pub/Sub

Bài viết được dịch từ [nguồn](https://medium.com/gitconnected/event-driven-systems-a-deep-dive-into-pubsub-architecture-39e416be913c)

Chào mừng bạn đọc đã đến với thế giới của kiến trúc pub/sub.

Trong bài viết lần này tôi sẽ giúp bạn đọc có một cái nhìn sâu hơn về cách thiết kế và xây dựng một hệ thống theo kiến trúc pub/sub.

Trước khi bắt đầu, hãy cùng nhau định nghĩa rõ hơn về khái niệm **kiến trúc pub/sub**. Nói một cách đơn giản thì đây là cách xây dựng hệ thống dựa trên mô hình **push data** & **pull data** giống như mô hình publisher-subscriber. Có nghĩa là một hoặc nhiều **publishers** có nhiệm vụ gửi dữ liệu cho một hoặc nhiều **subscribers** để các subscribers này tiến hành xử lí dữ liệu.

Pattern này được sử dụng rộng rãi ở nhiều loại hệ thống:

- Hệ thống phân tán (Distributed system).
- Hệ thống hướng sự kiện (Event-driven system).
- Hệ thống thời gian thực (Real-time system).

Kiến trúc pub/sub cho phép ta dễ dàng tách biệt các thành phần trong hệ thống với nhau cũng như dễ dàng thêm mới chức năng hoặc mở rộng hệ thống nếu cần.

Trong bài viết lần này, chúng ta sẽ cùng nhau tìm hiểu các loại hệ thống pub/sub khác nhau như:

- Push-based system.
- Pull-based system.
- Centralized system.
- Decentralized sytstem.

ngoài ra chúng ta cũng tìm hiểu thêm các công nghệ như Apache Kafka, RabbitMQ, ... Đồng thời tôi cũng sẽ chia sẻ với các bạn một vài best practices trong việc thiết kế cũng như triển khai hệ thống pub/sub, các vấn đề thường gặp và giải pháp cho chúng.

## Các loại hệ thống Pub/Sub

Đầu tiên đó là **push-based system**. Đây là hệ thống mà publisher sẽ gửi message đến cho subscriber bất kể subscriber có khả năng nhận message hay không. Điều này là rất tốt đối với các hệ thống cần thời gian "siêu thực" như "đầu tư chứng khoán" hoặc "dự báo thời tiết", ... Thế nhưng nhược điểm của nó là nó không quan tâm đến việc subscriber có đủ khả năng để xử lí lượng message được gửi đến hay không.

Ở chiều ngược lại ta có **pull-based system**, đây là hệ thống mà ở đó subscriber có khả năng lấy về message **khi bản thân subscriber ở trạng thái sẵn sàng**. Cơ chế này phù hợp cho các hệ thống mà ở đó subscriber cần có thời gian xử lí dữ liệu trước khi nhận thêm các messages mới, cơ chế này phù hợp cho các hệ thống **Image Recognition** hoặc **NLP** chứ không hề phù hợp với hệ thống thời gian thực.

![Screen Shot 2023-02-26 at 16 43 43](https://user-images.githubusercontent.com/15076665/221398449-9b2834d6-2dea-461d-963a-bd00926aea26.png)

Tiếp theo hãy cùng nhau nói về `centralized system` và `decentralized system`. Centralized system có một center hub hoặc server với vai trò làm trung gian giữa publisher và subscriber. Việc tồn tại server trung gian này sẽ giúp hệ thống dễ dàng điều khiển luồng dữ liệu hơn, tuy nhiên khi server hub bị quá tải, có thể dẫn đến tình trạng bottleneck.

Ngược lại thì `decentralized system` không hề có server hub, mà thay vào đó là sự tương tác `peer-to-peer` giữa các publisher và subscriber với nhau. Việc này giúp hệ thống dễ dàng được mở rộng, song lại rất khó để có thể điều khiển được luồng dữ liệu.

Vậy thì loại kiến trúc nào là sự lựa chọn tốt nhất? Câu trả lời đó là tuỳ theo yêu cầu hệ thống của bạn. Nếu hệ thống của bạn yêu cầu truyền dữ liệu thời gian thực và nhanh thì `push-based system` là một sự lựa chọn thích hợp.

Còn nếu như hệ thống của bạn cần phải có thời gian xử lí message trước khi nhận message tiếp theo thì `pull-based` là một sự lựa chọn phù hợp.

Một hệ thống dễ dàng kiểm soát và quản lí ? `Centralized system` là một sự lựa chọn tốt.

Còn nếu bạn cần một hệ thống có khả năng mở rộng cũng như tính uyển chuyển cao thì `Decentralized system` là thứ mà bạn cần.

> Trên thực tế, không có một sự lựa chọn nào là hoàn hảo cả, quan trọng hơn hết là ta cần hiểu sự khác biệt giữa các hệ thống để từ đó đưa ra sự lựa chọn "phù hợp" nhất.

## Ứng dụng của pub/sub architecture

Trong kiến trúc pub/sub, ta có publisher tạo ra message, subscriber sẽ tiêu thụ message. Điều đặc biệt ở pub/sub architecture đó là việc publisher và subscriber không cần phải biết về sự tồn tại của nhau để có thể tương tác với nhau.

Ứng dụng phổ biến nhất của pub/sub đó là **hệ thống phân tán - distributed system**

Hãy thử tưởng tượng, hệ thống của bạn có một tập các servers, chúng cần phải tương tác qua lại với nhau để thực hiện một tác vụ nào đó. Nếu ứng dụng kiến trúc pub/sub ở đây, ta có thể thấy rằng các servers sẽ vừa đóng vai trò là publisher, vừa đóng vai trò làm subscriber. Điều này giúp ta đơn giản hoá việc tương tác giữa các thành phần trong cùng một hệ thống với nhau, từ đó có thể dễ dàng "loại bỏ" cũng như "thêm mới" servers.

Một lĩnh vực khác mà pub/sub cũng có tính ứng dụng khá cao đó là **Event-driven system**. Trong hệ thống hướng sự kiện - event-driven system, các thành phần của hệ thống có thể subscribe một sự kiện nào đó, khi sự kiện đó xảy ra, chúng sẽ có những xử lí tương ứng phù hợp.

Thế nhưng một trong số những ứng dụng hữu ích nhất của pub/sub đó là trong **hệ thống thời gian thực - real time system**.
