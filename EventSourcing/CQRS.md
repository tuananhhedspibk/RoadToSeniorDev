# Tổng quan về CQRS

※ Bài viết được lược dịch từ <https://levelup.gitconnected.com/what-is-cqrs-8ddd74ca05bb>

## Tổng quan

`CQS` (Command Query Separation) design pattern hướng đến việc chia các methods thành 2 nhóm chính:

1. **Command**: loại này sẽ **thay đổi dữ liệu của object** và **không trả về gì** hoặc là **chỉ trả về meta-data**

2. **Query**: loại này sẽ trả về thông tin nhưng **không làm thay đổi dữ liệu của object**

Theo như lí thuyết nói trên, một method trong thực tế sẽ **không bao giờ đảm nhiệm vai trò của cả query và command**. Lấy một ví dụ tiêu biểu trong **Javascript**:

- method `push` là **command** khi nó đảm nhận việc đưa thêm phần tử mới vào stack
- method `top` là **query** khi nó đảm nhận việc trả về thông tin về phần tử trên cùng của stack
- method `pop` lại vi phạm nguyên tắc của **CQS** khi nó vừa **lấy ra phần từ trên cùng của stack** và **trả về thông tin của phần tử vừa lấy ra đó**

Do đó CQS nhấn mạnh vào việc "chia" một cách rõ ràng 2 thao tác "đọc" và "ghi". Việc này sẽ có ý nghĩa rất nhiều trong thực tế khi việc thực thi "song song" các câu **queries** sẽ không gây ra hậu quả gì nghiêm trọng trong khi việc thực thi "song song" các câu **commands** có khả năng sẽ gây ra sự bất đồng bộ (không nhất quán) về mặt dữ liệu.

Về mặt kĩ thuật, ta có thể áp dụng CQRS cho HTTP, với việc các API sẽ được triển khai:

- Bằng method **POST** cho **command**.
- Bằng method **GET** cho **Query**.

![Screen Shot 2023-09-16 at 12 59 13](https://github.com/tuananhhedspibk/SystemDesignInterview-P1/assets/15076665/a4ced43b-d6b2-4d85-8386-144d8bc2850e)

> Nhưng lúc này ngữ nghĩa đã chuyển qua phía URL

Ta lấy ví dụ với method POST để tạo một order mới sẽ có route là `/submit-order`, điều này trái ngược hoàn toàn với REST (nó chỉ cho phép đông từ của HTTP sẽ mô tả mục đích của API endpoint)

Đi xa hơn nữa, ta sẽ chia từ một DB thành 2 DBs riêng biệt:

- Một phục vụ cho việc đọc (có thể "phi chuẩn hoá - denormalizing" để tối ưu thao tác đọc).
- Một phục vụ cho việc ghi (cần phải "chuẩn hoá - normalizing")

Tuy nhiên cần có một cơ chế để đảm bảo sự "nhất quán" về mặt dữ liệu giữa 2 DBs này để từ đó:

- Vừa đảm bảo hiệu năng
- Vừa tăng tính sẵn sàng

cho hệ thống như hình minh hoạ dưới đây

![Screen Shot 2023-09-16 at 13 05 35](https://github.com/tuananhhedspibk/SystemDesignInterview-P1/assets/15076665/6fc501b4-7a13-4f9c-80c6-8248b6d4ded6)

## Dev API với CQRS

Khi dev API với CQRS, mọi chuyện không chỉ đơn thuần là chia routes thành POST và GET, song song với đó ta cũng cần đảm bảo rằng command **không trả về bất cứ gì (cùng lắm chỉ là meta-data mà thôi)**.

Cũng tương tự như với Query API, URL path sẽ mô tả câu query mà ta muốn thực hiện. Với trường hợp của query ta có thể truyền parameters thông qua query string (do nó là GET request), đồng thời cũng cần "phi chuẩn hoá" DB để có thể tối ưu được câu query.

Có một vấn đề khác đó là việc khi không tiến hành thực thi các query routes thì client không thể viết được command đã được thực hiện hay chưa, do đó ta có thể sử dụng các "third APIs" - Events API sẽ thông báo qua các events bằng việc thực thi push notifications thông qua `web socket` hoặc `HTTP streaming`, ...

Trong `GraphQL` ta có 3 khái niệm:

- Query
- Mutation
- Subscription

3 khái niệm này lần lượt tương ứng với

- Query
- Command
- Event API

trong CQRS do đó `GraphQL` cũng có thể coi là một công cụ lí tưởng để triển khai CQRS cho APIs.

## CQRS, DDD và Event-Sourcing

CQRS thường dược sử dụng kết hợp với **DDD** và **Event-Sourcing**. 3 khái niệm này tuy là khác nhau nhưng chúng lại bổ sung cho nhau.

Khi một command được gửi tới command API của CQRS-based appllication, nó cũng sẽ được coi là một **command cho aggregate** trong DDD. Aggregate sau đó sẽ **tạo ra một hoặc nhiều domain events**, các events này sẽ được **lưu trữ trong một event store** bằng Event-Sourcing và được sử dụng sau đó.

Ngoài ra các domain events được tạo ra trong quá trình cũng có thể sẽ được **chuyển tiếp** đến các events API để từ đó giúp kết nối hệ thống hiện thời với các hệ thống khác.

Không những thế domain events cũng sẽ được chuyển tiếp tới các read page khác trong ứng dụng để update view. Do đó có một sự tương ứng giữa **technical domain event** và view, các events sẽ tác động lên các view với sự trợ giúp từ các hành động CRUD.

## Quy tắc CAP và sự thống nhất sau cùng

Quy tắc CAP là viết tắt của:

- Consistency: tính thống nhất.
- Availability: tính sẵn sàng.
- Partition Tolerance: khả năng tiếp tục hoạt động ngay cả khi có lỗi.

Một hệ thống phân tán đảm bảo được tính thống nhất (consistency) khi thao tác ghi phải được phản ánh lên toàn bộ các nodes trước khi trả về cho phía client, để từ đó mọi nodes trong hệ thống sé đều trả về một response duy nhất.

Tính sẵn có (availability) mô tả việc một hệ thống có khả năng đáp ứng mọi write & read request tại mọi thời điểm, sẽ không bao giờ cho waiting time hoặc reject requests đến từ phía hệ thống.

Partition Tolerance đảm bảo việc hệ thống vẫn sẽ hoạt động ngay cả khi có nodes fail hoặc mất kết nối giữa các nodes.

Trên thực tế các hệ thống phân tán chỉ có thể đảm bảo được 2 trong số 3 tính chất nêu trên, tức là hoặc là CA, hoặc là CP hoặc là AP chứ không thể có cả 3.

Việc thiếu đi tính nhất quán (consistency) được coi như là nguy hiểm nhất.

## Tại sao lại sử dụng CQRS ?

Một trong những lí do tiêu biểu đó là nó bổ sung qua lại với các concepts khác như DDD hay Event-Sourcing và GraphQL

Việc CQRS chia tách rõ ràng giữa reading và writing cũng sẽ đem lại nhiều ưu điểm cho **kiến trúc microservices**.

Ngoài ra việc kết nối tốt với DDD và Event-sourcing sẽ giúp việc chia tách giữa **technical code** và **business code** trở nên rõ ràng và dễ dàng hơn.

Một nhược điểm thấy rõ nhất của CQRS đó là việc nó khá "phức tạp" thế nhưng hiẹn nay có rất nhiều framework hỗ trợ rất mạnh cho CQRS như NestJS (typescript, nodejs)

## Tổng kết

CQRS là một cách tiếp cận khá thú vị cho các hệ thống phân tán để từ đó có thể tận dụng được những ưu điểm của DDD và Event-sourcing. Dù rằng việc triển khai CQRS là phức tạp hơn so với kiến trúc client-server truyền thống nhưng bù lại ứng dụng lại có khả năng mở rộng cao trong thương lai.

Cảm ơn các bạn đã đọc bài viết này của tôi về CQRS hi vọng rằng bạn đọc sẽ có được cho mình cái nhìn tổng quan cũng như nắm được những nội dung cơ bản về CQRS.

Hẹn gặp lại ở các bài viết tiếp theo, xin trân thành cảm ơn.

