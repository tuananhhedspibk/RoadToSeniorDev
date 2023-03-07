# Làm cách nào để kết hợp DDD, Hexagonal, Onion, Clean, CQRS

Bài viết được dịch từ [nguồn](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/#domain-layer)

Bài viết này là một phần của [Biên niên sử về Software Architecture](https://herbertograca.com/2017/07/03/the-software-architecture-chronicles/), nằm trong [một series các bài viết về Software Architecture](https://herbertograca.com/tag/software-architecture/page/2/). Trong những bài viết đó, tôi có viết về những gì mình đã học được về Software Architecture, cách nghĩ của tôi về chúng và cách tôi sử dụng những kiến thức đó. Nội dung của bài viết này sẽ có nghĩa hơn nếu bạn đã đọc những bài viết trước đó trong series nói trên.

## Các thành phần cơ bản của một hệ thống

Tôi sẽ bắt đầu từ [EBI](https://herbertograca.com/2017/08/24/ebi-architecture/) và [Ports & Adapters](https://herbertograca.com/2017/09/14/ports-adapters-architecture/) architectures. Cả hai kiến trúc này đều phân biệt một cách tường minh những gì thuộc về `internal` và những gì thuộc về `external` và những gì dùng để kết nối `internal code` & `external code`.

Hơn nữa, **Port & Adapters** cũng định nghĩa 3 thành phần cơ bản của một hệ thống:

- **User interface**
- **Business logic** hoặc **application core** - được sử dụng bởi UI để thực hiện các nghiệp vụ.
- **Infrastructure** - kết nối application core với các tooles như DB, hoặc search engine hoặc APIs của bên thứ 3.

![000-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/222963258-87af8200-965d-4790-b438-fc5662f1ff85.png)

Application core chính là cái mà chúng ta nên dành sự quan tâm cao nhất. Nó chính là thành phần cho phép code của chúng ta thực hiện những nghiệp vụ chính của hệ thống. Nó có thể sử dụng một vài user interface (CLI, API, ...) nhưng chức năng của nó vẫn vậy và nó không cần phải quan tâm xem phía user interface trigger nó như thế nào.

Như bạn có thể tưởng tượng, flow điển hình của application đó là đi từ user interface, qua application core, đến infrastructure code, quay ngược trở lại application core và cuối cùng trả về response cho phía user interface như hình minh hoạ bên dưới.

![010-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/222990715-2c023aae-6eb3-4985-b26c-5e0217f15a4e.png)

## Tools

Đi xa hơn so với application core, chúng ta có tools mà ứng dụng sẽ sử dụng như database engine, search engine, Web server hoặc CLI console.

![020-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/222990793-837a42b8-4177-4fb6-987d-379041e1c4f7.png)

Việc đặt CLI console vào chung một "bucket" với DB engine trông có vẻ khá kì cục thể nhưng trong thực tế nó lại là tools được sử dụng bởi ứng dụng.

Điểm khác biệt chính ở đây đó là trong khi CLI console và web server được sử dụng để **ra lệnh cho ứng dụng làm một điều gì đó**, database engine **được ra lệnh bởi ứng dụng là phải làm một điều gì đó**.

### Kết nối tools và đưa các cơ chế vào Application core

Code units kết nối tools với application core được gọi là `adapters`. Adapters triển khai code cho phép business logic tương tác với một tool nào đó và ngược lại.

Adapter **ra lệnh** cho ứng dụng làm một việc gì đó được gọi là **Primary hoặc Driving Adapters** trong khi adapter **được ra lệnh** bởi ứng dụng sẽ được gọi là **Secondary hoặc Driven Adapters**.

### Ports

Bản thân các Adapters không hề được tạo ra một cách ngẫu nhiên, chúng được tạo ra với mục đích khớp với các đầu vào của Application Core, ở đây là **Port**. Port giống như một **đặc tả** về cách tool sử dụng application core hoặc cách mà application core sử dụng tool.

Trong thực tế thì các Port này sẽ là:

- Interface
- Tổ hợp các interfaces
- DTOs

Một điều quan trọng ở đây đó là **Ports (interfaces) thuộc về bên trong business logic** trong khi Adapter sẽ nằm bên ngoài business logic.

### Primary hoặc Driving Adapters

Primary (Driving) Adapter sẽ bao ngoài port và "ra lệnh" cho Application core "làm việc". Adapter sẽ có nhiệm vụ **chuyển hoá tất cả từ delivery mechanism thành method call trong Application Core.**

![030-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/223136385-c203417b-fe0e-42d6-af2d-b7d955a99e8d.png)

Hay nói cách khác Driving Adapters là Controllers hoặc Console commands, các Adapters này sẽ inject các objects (là instance của các class implement các port - interface mà controller cần).

Nói một cách cụ thể hơn thì Port ở đây có thể là `Service Interface` hoặc `Repository Interface` mà controller cần tới. Các implementations của Port sau đó sẽ được inject vào controller.

Ngoài ra thì port cũng có thể là `Command Bus Interface` hoặc `Query Bus Interface`.

### Secondary hoặc Driven Adapters

Không giống như Driving Adapter (sẽ wrap lấy ports) thì `Driven Adapter` sẽ **implement port** - interfaces được inject vào Application core

![040-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/223280997-7084d53c-f739-424a-80aa-fbe26c288a55.png)

Ta lấy ví dụ, giả sử ta có một ứng dụng cần lưu và xoá dữ liệu. Ta sẽ tạo ra một interface đáp ứng các nhu cầu trên với 2 methods là `save` và `delete`. Application khi cần lưu hoặc xoá dữ liệu chỉ cần "yêu cầu" object là instance của class implement interface đó trong constructor của mình.

Giả sử ban đầu, chúng ta sử dụng MySQL để quản lí DB, nhưng sau đó ta lại muốn chuyển sang PostgreSQL hoặc MongoDB, thì khi này ta chỉ cần implement interface ở trên, sửa đổi `Driven Adapter` để nó sử dụng đúng các hàm quy định sẵn của PostgreSQL hay MongoDB, sau đó inject vào constructor của application core là được.

### Inversion of control

Một trong những chú ý về pattern này chính là adapters phụ thuộc vào tool và port cụ thể (bởi vì chúng sẽ implement port - interface) nhưng business logic chỉ phụ thuộc vào port (interface) mà thôi, bản thân port cũng được thiết kế để phù hợp với những gì mà business logic cần, nên bản thân port cũng sẽ không phụ thuộc vào một adapter hay tool cụ thể nào cả.

![050-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/223447285-c565b92b-51d1-44b8-b5ff-5b42dcfbcb9f.png)

Điều này đồng nghĩa với việc mối quan hệ phụ thuộc sẽ hướng vào trung tâm của hệ thống - đây được gọi là **inversion of control** ở level kiến trúc hệ thống.

Một điều nữa cũng cần phải chú ý ở đây đó là:

> Ports được tạo ra để đáp ứng nhu cầu của Application Core chứ không chỉ thuần tuý là mô phỏng lại tools APIs.

## Tổ chức Application Core

### Application Layer

Usecase là một "tiến trình" - nó sẽ được trigger bên trong `Application Core` của hệ thống với một vài yếu tố như User Interface phía bên ngoài, ...

Usecase được định nghĩa bên trong Application Layer - đây chính là tầng đầu tiên được cung cấp bởi DDD và được sử dụng bởi Onion Architecture.

![060-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/223448761-a88aeb6d-8a65-4824-abe8-2dac492c7429.png)

Tầng này sẽ bao hàm:

- Application Services (và interfaces của chúng) - như một dạng "công dân loại một".
- Port & Adapters interfaces (có thể bao hàm ORM interfaces, search engine interfaces, messaging interfaces).
- Có thể chứa các Handlers cho Command và Query Buses.

Application Services bao hàm các logic triển khai usecase. Cụ thể nhiệm vụ của chúng là:

1. Sử dụng repository để tìm một hoặc một vài entities.
2. Yêu cầu các entities thực hiện domain logic.
3. Sử dụng repository để lưu các entities hoặc lưu data changes.

Command Handlers có thể được sử dụng theo 2 hướng khác nhau:

1. Chúng có thể bao hàm các logic để thực thi usecase.
2. Chúng cũng có thể sử dụng như những sợi dây "truyền tin" trong hệ thống, nhận Command và trigger logic nằm trong Application Service.

Việc lựa chọn cách tiếp cận nào là tuỳ vào hoàn cảnh của chúng ta:

- Liệu chúng ta đã có sẵn Application Service và giờ chỉ cần thêm Command Bus ?
- Liệu rằng Command bus có cho phép chỉ định class/ method nào đó như là handler hoặc mở rộng thêm các classes hoặc interfaces có sẵn nhằm phục vụ mục đích của mình.

Tầng này cũng có thể bao hàm việc trigger **Application Events**. Các events trigger logic này chính là side-effect của usecase, ví dụ như:

- Gửi email.
- Gửi thông báo tới API của bên thứ 3.
- Push notification.
- Hoặc khởi động / gọi đến usecase thuộc về một component khác trong ứng dụng.

### Domain Layer

Các objects thuộc tầng này bao hàm các logic chỉnh sửa dữ liệu tuỳ theo từng domain logic khác nhau. Nó độc lập so với usecase (business logic).

![070-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/223453897-a098087f-83c7-4997-b281-3fc9f32f4b10.png)

#### Domain Services

Đôi khi chúng ta sẽ thấy những domain logic gọi đến nhiều entities nhưng lại không thuộc về một entities cụ thể nào cả, chúng ta sẽ cảm giác rằng logic đó không thuộc về trách nhiệm của bất kì một entities nào cả.

Do đó phản xạ đầu tiên của chúng ta chính là đưa chúng vào tầng `Application Service`, thế nhưng trong thực tế thì việc đưa chúng sang một tầng khác sẽ làm cho các domain logic đó không thể được tái sử dụng ở các usecases khác nhau, do đó:

> Hãy giữ cho domain logic tránh xa khỏi application layer.

Giải pháp ở đây đó là tạo ra Domain Service, có chức năng nhận vào một tập các entities và thực thi một vài business logic trên chúng. `Domain service` thuộc về `Domain layer` do đó chúng không hề biết gì về các classes thuộc về tầng Application như `Application Services` hay `Repositories`.

Ngoài ra thì `Domain Service` cũng có thể sử dụng các `Domain services` khác cũng như các `Domain Model objects` khác.

#### Domain Model

Nằm sâu ở bên trong trung tâm hệ thống, không phụ thuộc vào bất cứ thứ gì trừ nó - sẽ bao hàm business objects biểu diễn các domain logic, ví dụ tiêu biểu ở đây đó là `Entities`, `Value-Objects`.

Domain model cũng chính là nơi **Domain events** tồn tại, các events này được kích hoạt khi một tập dữ liệu nhất định bị thay đổi. Nói cách khác khi entity thay đổi, Domain Event được kích hoạt và nó quan tâm đến sự thay đổi giá trị của các thuộc tính. Events này sẽ rất có hữu ích đặc biệt là khi được sử dụng trong `Event Sourcing`.

## Components

Ở các phần trên chúng ta đã đề cập đến việc chia tách code dựa theo layers. Nhắc tới việc chia tách code, chúng ta còn có chia tách theo tính năng (Package by feature) hoặc chia tách theo component (Package by component) và chia tách theo layer (Package by layer) như trên.

Những cách phân chia trên được giải thích rất cụ thể trong blog [Package by component and architecturally-aligned testing](http://www.codingthearchitecture.com/2015/03/08/package_by_component_and_architecturally_aligned_testing.html) của tác giả Simon Brown.

![Screen Shot 2023-03-08 at 8 34 23](https://user-images.githubusercontent.com/15076665/223579794-cf827357-7c50-4544-ba24-9a40857f2d08.png)

Cá nhân tôi thì thích "Package by component" hơn, dưới đây là sơ đồ cho cách tiếp cận này được tôi lấy ra từ bài viết của Simon Brown.

![Packaging 4_3](https://user-images.githubusercontent.com/15076665/223580076-9314708a-e7af-4a3c-a064-ca77f68727bd.png)
