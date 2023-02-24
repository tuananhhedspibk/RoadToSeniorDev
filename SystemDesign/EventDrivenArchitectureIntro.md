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

- **Events xảy ra ở những thời điểm khác nhau**
- **Events tồn tại dưới hình thức records**. Events và records là hai thuật ngữ khác nhau về mặt kĩ thuật. Event dùng để chỉ một hành động hoặc một điều gì đó xảy ra. Record chứa thông tin mô tả về event. Ở đây chúng ta thường sử dụng thuật ngữ *event* để chỉ đến record của nó.
- **Producers phát hiện các events bằng cách publish các records tương ứng với chúng vào stream**.
- **Stream lưu trữ các chuỗi records**. Stream có thể được tạo nên dựa trên nền tảng là các disk-based **log** hoặc cũng có thể là **database tables**, ...
- **Brokers quản lí truy cập đến streams**, theo dõi các thao tác đọc và ghi (reading & writing), xử lí consumer state, cũng như quản lí các tasks trên streams. Ví dụ: broker có thể cắt bỏ bớt các contents trên stream khi stream trở nên quá tải với lượng records hiện tại.
- **Consumer đọc từ streams và xử lí records**. Việc xử lí event có thể dẫn đến các side-effect như: chèn thêm entity vào database hoặc tái cấu trúc state của một remote entity, ...
- **Vai trò của consumers và producers có thể chồng chéo nhau**. Ví dụ, việc consumer xử lí một event có thể dẫn tới việc phát sinh một event khác.

## Decoupling thông qua bất đồng bộ và tổng quát hoá

Quay trở về điểm khởi đầu, *Tại sao EDA giúp các thành phần trong hệ thống giảm phụ thuộc vào nhau ?*

Một định nghĩa đơn giản về độ phụ thuộc đó là **mức độ một component làm ảnh hưởng đến các components khác**.

Một ví dụ tiêu biểu đó là khi một service gọi một REST API khác và chờ kết quả trả về từ API đó, nếu trong trường hợp REST API kia gặp trục chặc và không trả về kết quả được thì service gọi đến API sẽ phải dừng mọi xử lí phía sau lời gọi API đó.

Ta nói rằng các components là *tightly coupled* nếu chúng phụ thuộc chặt chẽ vào nhau và *loosely coupled* trong trường hợp ngược lại.

![Screen Shot 2023-02-12 at 21 44 40](https://user-images.githubusercontent.com/15076665/218311996-26469db4-0e51-4a01-80fc-c4a1cc368790.png)

1. Nhắc lại là events *chỉ xảy ra*. Event procuder không hề biết đến sự tồn tại của các components khác. Do đó producer vẫn sẽ làm việc ngay cả khi consumer không hoạt động, thế nên bản thân broker cũng sẽ định kì buffer các event records mà không gặp phải "sự thúc bách" nào từ phía producer.
2. Việc lưu trữ các event records trong broker giúp xoá tan đi định nghĩa về mặt thời gian. Cụ thể là producer có thể publish event ở thời điểm *T1*, consumer sẽ "tiêu hoá" event đó ở thời điểm *T2*, về cơ bản *T1* và *T2* sẽ chênh lệch nhau vài milisecond (nếu mọi thứ hoạt động tốt) hoặc vài giờ (nếu một vài consumers gặp trục trặc).

EDA không phải là "viên đạn bạc" hay "một liều thuốc tiên" giúp ta có thể tách bạch các components ra khỏi nhau hoàn toàn. Trên thực tế khi producer và consumer không phụ thuộc vào nhau nữa thì chúng lại **phụ thuộc vào broker**, dẫn đến một **điểm lỗi** mới đó chính là broker, nên broker phải đảm bảo **hiệu năng cao cũng như tính chịu lỗi tốt**.

## Các hình thứ xử lí event

Có thể phân ra thành 3 nhóm chính như dưới đây:

### Xử lí event rời rạc (Discrete event processing)

Ví dụ như khi post một bài đăng lên mạng xã hội. Một trong số đặc trưng của việc xử lí event rời rạc đó là sự hiện diện của một event không hề liên quan đến các events khác và hoàn toàn có thể được xử lí độc lập.

### Event stream processing

Xử lí các event theo luồng, có tính đến thứ tự của các events, khi mà event hiện tại có liên quan đến event trong quá khứ. Một ví dụ tiêu biểu đó là các events thay đổi lên business entity. Các events này sẽ được xử lí theo một thứ tự nhất định, sau đó sẽ lưu business entity data vào trong database. Consumer cũng cần tránh tình trạng đồng thời thay đổi lên cùng một record của database khiến cho dữ liệu không được nhất quán.

### Complex event processing

Complex event processing (CEP) định danh và đưa ra các event pattern phức tạp dựa trên một chuỗi các event đơn giản. Ta lấy ví dụ về CEP cho việc theo dõi nhiệt độ và khói từ các cảm biến để phát hiện xem có xảy ra hoả hoạn hay không. Dữ liệu về nhiệt độ ở một thời điểm có thể không mang quá nhiều ý nghĩa nhưng với một "cụm nhiệt độ" cũng với tỉ lệ thay đổi nhiệt độ thì ta hoàn toàn có thể phán đoán được rằng liệu có đang xảy ra hoả hoạn hay không.

## Khi nào thì sử dụng EDA (Event Driven Architecture)

Có một vài use-cases tiêu biểu như sau:

1. **Opaque consumer ecosystem**: khi mà producers hầu như không biết gì về consumers.
2. **High fan-out**: kịch bản khi mà một event có thể được xử lí bởi nhiều consumers khác nhau.
3. **Complex pattern matching**: khi mà các events có thể được "xâu chuỗi" lại để đưa ra các events phức tạp hơn.
4. **Command-query responsibility segregation**: CQRS là một pattern nhằm phân chia các thao tác đọc & ghi lên data store. Việc triển khai CQRS sẽ giúp hệ thống có khả năng mở rộng cao hơn cũng như tăng tính tin cậy cho hệ thống, đổi lại sẽ là sự nhất quán trong dữ liệu có thể sẽ không được hoàn chỉnh như mong muốn. Pattern này thường đi đôi với EDA.

## Lợi ích của EDA

1. **Buffering & fault-tolerance**: Events có thể được xử lí ở các mức độ khác nhau tuỳ theo ứng dụng và producer không cần phải "tự làm chậm mình" để consumer có thể đuổi kịp.

2. **Decoupling producer & consumer**: giảm sự phụ thuộc lẫn nhau giữa producer và consumer. Từ đó ta có thể dễ dàng thêm hoặc bớt consumers và producers vào hệ thống.

3. **Dễ dàng scale**: chúng ta có thể dễ dàng phân các events vào các substream khác nhau và xử lí chúng một cách song song cũng như thêm các consumers để có thể kịp thời xử lí số lượng lớn các events.

## Nhược điểm của EDA

1. **Giới hạn trong việc xử lí bất đồng bộ**: EDA là một pattern khá mạnh trong việc giảm sự phụ thuộc lẫn nhau giữa các components, xong nó lại gặp ít nhiều hạn chế với xử lí bất đồng bộ.

2. **Phát sinh thêm những vấn đề phức tạp mới**: với mô hình client-server, request-response truyền thống ta chỉ cần có **2 nhân tố chính** mà thôi, trong khi với EDA ta còn cần thêm cả **broker** để có thể xử lí việc tương tác giữa **consumer** và **producer**.

3. **Failure masking**: với các hệ thống có sự phụ thuộc lẫn nhau, khi có một component nào đó gặp lỗi, nó sẽ làm ảnh hưởng đến các components khác vì các components khác này cũng sẽ biết rằng có component gặp lỗi (điều này là hoàn toàn không tốt khi một component làm ảnh hưởng đến toàn bộ hệ thống), thế nhưng với EDA khi các component không phụ thuộc lẫn nhau, việc 1 component gặp lỗi có thể sẽ không được các components khác biết đến. Điều này vô tình làm cho lỗi đó không được biết đến và xử lí (dù rằng nó không làm ảnh hưởng đến các components khác). Do đó với EDA ta cần có một cơ chế **logging** và **monitoring** với mỗi **event-driven component**, nhưng điều này sẽ làm tăng độ phức tạp của hệ thống.

## Những điều cần lưu ý

EDA không hoàn hảo 100%, cũng như các công cụ khác, nó cũng có nhược điểm riêng. Dưới đây là những nhược điểm của EDA mà các developers cũng như architecture nên chú ý khi thiết kế cũng như triển khai hệ thống hướng sự kiện (event-driven system).

1. **Convoluted choreography**: việc làm giảm sự phụ thuộc lẫn nhau của các components có thể làm cho kiến trúc của hệ thống giống như **Rube Goldberg machine** khi mà business logic sẽ được triển khai bằng một chuỗi các **side-effect** được "nguỵ trang" dưới mác "event": một component đưa ra event, kích hoạt xử lí của component khác, component khác này lại đưa ra event và kích hoạt một component khác nữa, và cứ như thế ... Cách tương tác này giữa các components sẽ nhanh chóng trở nên khó hiểu và kiểm soát.

2. **Nguỵ trang commands dưới mác sự kiện**: event là chỉ thuần tuý mô tả lại một điều gì đó vừa mới xảy ra chứ nó không nói về việc sự kiện hoặc một điều gì đó nên được xử lí như thế nào. Hay nó cách khác, **command** - **chỉ thị** là một chỉ dẫn trực tiếp cho một component cụ thể nào đó. Do **commands** & **events** đều là các message ngắn nên dễ có sự hiểu nhầm rằng command chính là event.

3. **Khó đoán định được consumers muốn gì**: event chỉ nên chứa các thông tin cần thiết cho việc nó được xử lí ra sao. Nhưng trên thực tế chúng ta có thể thêm những thông tin "thừa thãi" khác vào event.

## Tổng kết

Kiến trúc microservices là một mảnh ghép cho quá trình xây dựng một hệ thống dễ dàng bảo trì, mở rộng hơn. Microservice thực sự tuyệt khi đứng ở phương diện tách các components khỏi nhau nhưng nó cũng có rất nhiều vấn đề nổi cộm. Việc chia nhỏ hệ thống từ "monolith" thành "microservice" có thể khiến chúng ta quay lại đúng nới chúng ta bắt đầu, đó chính là vấn đề "distributed monolith" - các monolith phân tán.

Để hoàn thành mảnh ghép dang dở và giải quyết vấn đề phụ thuộc lẫn nhau giữa các components, chúng ta tìm kiếm đến kiến trúc hướng sự kiện - event-driven architecture.

EDA là một công cụ tốt để phân tách các components trong hệ thống bằng cách mô hình hoá sự tương tác giữa chúng thông qua việc sử dụng các khái niệm *producers*, *consumers*, *events*, *streams*. Event mô tả một điều gì đó vừa mới xảy ra, nó được sinh ra và xử lí một cách bất đồng bộ bởi các components không hề biết gì về nhau. EDA cho phép các components hoạt động độc lập với nhau. Bản thân EDA không phải là hoàn hảo, nhưng nó đem lại nhiều lợi ích hơn là vấn đề. Do đó EDA có thể được xem như một thành phần không thể thiếu của bất cứ hệ thống microservices thành công nào.
