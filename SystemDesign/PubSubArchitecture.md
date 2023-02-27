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
Với pub/sub ta có thể xây dựng được một hệ thống **gần thời gian thực** có khả năng xử lí dữ liệu và response gần như ngay tức thì.

## Triển khai kiến trúc pub/sub

Có rất nhiều công cụ để triển khai kiến trúc pub/sub. Trong phần này, tôi sẽ đi sâu vào một vài công cụ phổ biến nhất để triển khai kiến trúc pub/sub và giúp bạn có thể đưa ra quyết định lựa chọn công cụ thích hợp cho mình.

Đầu tiên chúng ta có `Apache Kafka`, đây là công cụ phổ biến trong những năm gần đây với khả năng mở rộng cao cũng như có thể xử lí một số lượng lớn dữ liệu trong thời gian thực. Hơn nữa nó cũng được "thực chiến" bởi các công ty lớn như Uber hay Netflix.

Tiếp theo chúng ta có `RabbitMQ`. Open-source message broker này được xây dựng trên Advanced Message Queuing Protocol (AMQP). Nó khá phù hợp khi bạn mới triển khai pub/sub architecture vì dễ dàng thiết lập và quản lí.

Cuối cùng chúng ta có `Google Cloud Pub/Sub`, đây là dịch vụ được triển khai bằng Google Cloud, cho phép chúng ta có thể gửi và nhận message giữa các ứng dụng độc lập với nhau. Nó được quản lí bởi Google nên ta không cần phải quan tâm đến infrastructure.

Vậy, dựa theo yêu cầu về tính mở rộng của hệ thống bạn có thể có những sự lựa chọn sau:

- Nếu hệ thống cần xử lí nhiều dữ liệu và nhiều users, hãy lựa chọn `Kafka`.
- Nếu hệ thống mới chỉ bắt đầu và không có quá nhiều traffic, hãy lựa chọn `RabbitMQ`.

Một yếu tố quan trọng khác cũng cần được cân nhắc ở đây đó là `tính tin cậy`. Bạn cần đảm bảo hệ thống vẫn giữ được messages ngay cả khi nó bị "sập". Về điều này thì Google Cloud là một ứng cử viên sáng giá do hệ thống infra của nó được Google quản lí 100%.

Cuối cùng đó là hiệu năng, nếu bạn cần hệ thống có khả năng xử lí dữ liệu bất kì lúc nào mà không bị "lag", hãy nghĩ đến `Kafka` - vì bản thân `Kafka` được thiết kế để xử lí một lượng lớn dữ liệu trong thời gian thực.

> Tóm lại hãy xem xét đến khả năng mở rộng, tính tin cậy và hiệu năng để có thể đưa ra một sự lựa chọn tốt cho việc triển khai pub/sub của bạn

## Best practices & Challenges

Đầu tiên, hãy nắm rõ yêu cầu hệ thống. Hệ thống có cần xử lí thời gian thực hay không? Có yêu cầu cao về khả năng mở rộng và tính tin cậy hay không? Trả lời được những câu hỏi trên sẽ giúp bạn tìm ra được công cụ phù hợp cho hệ thống của mình.

Tiếp theo hãy chú ý đến `messaging protocol`. Đây chính là ngôn ngữ mà pub/sub system sẽ sử dụng để tương tác. Ví dụ: nếu bạn đang cần xử lí binary data thì protocol như Google Protocol Buffer sẽ hữu ích hơn so với text-based protocol như JSON.

Một điều nữa cần phải lưu ý đó là giữ cho pub/sub system của bạn "tách bạch" nhất có thể. Có nghĩa là các thành phần trong hệ thống của bạn cần phải hoạt động độc lập với nhau để có thể dễ dàng mở rộng và bảo trì. Điều này có thể được thực hiện thông qua `message queues` - hoạt động như một buffer giữa publisher và subscriber.

Giờ hãy nói về một vài vấn đề mà bạn sẽ gặp phải với kiến trúc pub/sub. Một trong những vấn đề lớn nhất đó là `message loss` - có thể message được gửi đi nhưng không ai nhận được, hoặc message được nhận nhưng lại không được xử lí. Để giải quyết điều này ta cần:

- Một hệ thống xử lí lỗi mạnh.
- Một hệ thống monitoring các message bị mất.

Một vấn đề khác đó là `data-stream với tốc độ cao`. Nhận quá nhiều dữ liệu sẽ làm cho việc xử lí chúng theo đúng thứ tự thời gian nhận được là rất khó. Hãy cân nhắc đến các kĩ thuật như `data buffering` hoặc `data sampling`.

Vấn đề cuối cùng đó chính là `hiệu năng` của hệ thống. Khi hệ thống của bạn được mở rộng thêm thì đồng nghĩa với việc độ phức tạp cũng sẽ tăng lên, điều đó có thể làm cho hiệu năng đi xuống. Hãy liên tục monitoring hệ thống của bạn và có những thay đổi phù hợp nhất có thể.
