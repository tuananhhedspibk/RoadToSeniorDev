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

Các đoạn code sẽ chia components theo từng layers một như hình minh hoạ phía trên. Ta thấy rằng, components có thể là  Billing, User, Review hoặc Account nhưng chúng luôn luôn liên quan đến domain. `Bounded context` như `Authorization` hoặc `Authentication` nên được xem như các `external tools` nhằm phục vụ khi chúng ta tạo các adapter và ẩn chúng phía sau các port.

![080-explicit-architecture-svg](https://user-images.githubusercontent.com/15076665/223730032-40dc4ce5-c3a0-4f07-8557-2232fbfc2036.png)

### Decoupling các components

Cũng tương tự như việc chia nhỏ code thành các đơn vị như (classes, interfaces, mixins, ...), thì việc giảm sự phụ thuộc lẫn nhau giữa các components cũng sẽ đem lại cho ta những lợi ích đáng kể.

Để chia tách các classes với nhau, ta có thể sử dụng `Dependencies Inject` - bằng cách inject các dependencies vào class thay vì thực thể hoá nó, làm class phụ thuộc vào sự trừu tượng (abstractions - interface hoặc abstract classes) thay vì các classes cụ thể. Do đó class tiến hành inject dependencies không hề biết gì về class triển khai (implement) interface hoặc abstract class đó cả.

Cũng tương tự như vậy, việc chia tách các components cũng sẽ làm cho các components này không hề biết gì về nhau. Điều đó cũng đồng nghĩa rằng Dependency Injection và Dependency Inversion không đủ để chia tách các components do đó ta cần cấu trúc lại kiến trúc. Chúng ta có thể cần đến shared kernel, eventual consistency và thậm chỉ cả discovery service.

![100 - Explicit Architecture](https://user-images.githubusercontent.com/15076665/223735538-7f32bc39-4083-40ed-ab96-23dc0e5212e9.png)

### Trigger logic trong các components khác

Khi một component của chúng ta (component B) cần thực hiện một tác vụ nếu một sự kiện nào đó xảy ra ở trong component A, chúng ta không thể tiến hành gọi trực tiếp từ component B sang component A vì khi đó hai components này sẽ phụ thuộc lẫn nhau.

Thế nhưng chúng ta cũng có thể tiến hành cho component A dispatch ra một event nào đó và event listener bên trong component B sẽ lắng nghe event trên, khi nhận được event, nó sẽ tiến hành trigger các logic cần phải được thực hiện bên trong component B. Điều đó đồng nghĩa với việc component A sẽ phụ thuộc vào `event dispatcher` nhưng lại **tách biệt hoàn toàn** so với component B.

Ngoài ra, nếu bản thân event "tồn tại" trong A thì điều này có nghĩa rằng B sẽ biết về sự tồn tại của A nên nó sẽ phụ thuộc vào A. Để loại bỏ đi sự phụ thuộc này, chúng ta sẽ tạo ra một library với một tập các `application core functionality` sẽ được shared giữa các components - ta gọi đó là `Shared Kernel`. Điều này có nghĩa rằng các components sẽ phụ thuộc vào `Shared Kernel` nhưng lại "đứng độc lập" với nhau.

`Shared Kernel` sẽ bao gồm các tính năng như:

- Application events
- Domain events

Tuy nhiên chúng ta nên giữ cho `Shared Kernel` này "nhỏ" và "nhẹ" nhất có thể vì dù là bất kì sự thay đổi nào ở shared kernel này cũng sẽ làm ảnh hưởng đến toàn bộ các components phụ thuộc vào nó. Giả sử ta có một hệ thống đa ngôn ngữ, một hệ sinh thái micro-services đi chẳng hạn, mỗi một service sẽ được viết bằng một ngôn ngữ khác nhau, do đó shared kernel cần phải được viết bằng một thứ ngôn ngữ mà các services đều có thể hiểu được. Ta lấy ví dụ với `Event class`, nó sẽ chứa các thuộc tính như:

- Event name
- Event description
- ...

được viết bằng `JSON`, chính vì lí do đó nó có thể được đọc và thông dịch bởi các ngôn ngữ khác nhau của các services. Bạn có thể đọc thêm trong bài viết sau đây của tôi: [Hơn cả concentric layers](https://herbertograca.com/2018/07/07/more-than-concentric-layers/)

![explicti_arch_layers](https://user-images.githubusercontent.com/15076665/223738656-3d040c59-19c4-45d0-83cf-3e3188978328.png)

Cách tiếp cận này hoạt động tốt với cả monolithic app và distributed app như micro-services ecosystem. Do event sẽ được phát sinh một cách bất đồng bộ nên việc trigger các logic nằm ở các components khác ngay lập tức là điều không thể.

Việc component A thực hiện lời gọi HTTP tới component B một cách trực tiếp sẽ làm cho 2 components này phụ thuộc lẫn nhau, do đó để làm cho chúng tách bạch, không liên quan gì đến nhau, ta có thể sử dụng `discovery service` - component A sẽ phải "hỏi" discovery service để biết được địa chỉ đích để gửi request đến đó, ngoài ra `discovery service` cũng có thể proxy các requests tới các services liên quan và sau cùng trả về các response tới requester.

Cách tiếp cận nêu trên có thể sẽ khiến cho các components phụ thuộc vào `discovery service` nhưng nó sẽ giúp cho các components không bị lệ thuộc vào nhau.

### Lấy dữ liệu từ các components khác

Như chúng ta đã thấy, việc một component thay đổi dữ liệu của component khác là điều không được phép. Thế nhưng việc component query và sử dụng dữ liệu từ một component khác là điều hoàn toàn có thể.

#### Data Storage share giữa các components

Ta lấy ví dụ `billing component` cần biết tên của client thuộc về `account component`, lúc này `billing component` cần gửi truy vấn dữ liệu đến `shared data storage` để lấy về dữ liệu mà nó cần. `Shared data storage` là tập hợp của nhiều dữ liệu khác nhau, thế nhưng các component khi sử dụng các dữ liệu này thì chỉ có thể sử dụng ở chế độ `read-only` mà thôi (nghĩa là sử dụng thông qua query).

#### Data Storage tách biệt theo từng component

Trong trường hợp này, mỗi component sẽ có riêng cho mình một storage riêng, mỗi storage sẽ bao gồm:

- Một tập dữ liệu thuộc về sở hữu của component và chỉ được chỉnh sửa bởi chính component đó.
- Một tập dữ liệu khác là bản sao dữ liệu của các components khác, component sở hữu tập này chỉ được phép query cho chức năng của mình chứ không được phép chỉnh sửa gì cả, nó cần được cập nhật khi component "chủ" thay đổi các dữ liệu sao này.

Mỗi component sẽ tạo ra cho mình một bản sao dữ liệu (dạng local) được sao chép từ các components khác. Khi các components khác thay đổi dữ liệu, bản thân component đó sẽ trigger một event, các components khác đang chứa bản sao dữ liệu sẽ lắng nghe event này (domain event) và cập nhật dữ liệu local của chúng.

## Flow of control

Như đã trình bày ở trên, flow của chúng ta sẽ là user → application core → user. Thế nhưng làm cách nào để các classes có thể "hoà nhịp" cùng với nhau ? Cái nào phụ thuộc vào cái nào ? Và làm thế nào để kết hợp chúng lại ?

### Khi không có Command/Query Bus

Trong trường hợp ta không sử dụng command bus, thì controller sẽ phụ thuộc hoặc là vào Application Service hoặc là Query Object.

![4_3 UMLish_1](https://user-images.githubusercontent.com/15076665/224045094-ef183984-b81b-433c-aa06-b711835e9330.png)

`Query Object` sẽ trả về các raw data cho phía user. Dữ liệu trả về dưới dạng DTO - sẽ được inject vào `ViewModel` - ViewModel có thể có một vài view logic bên trong nó.

`Appliation Service` - sẽ chứa các usecase logic, các logic này thường sẽ tiến hành chỉnh sửa dữ liệu thay vì chỉ view dữ liệu thuần tuý. Application Service sẽ phụ thuộc vào Repositories - trả về các entites bao gồm các logic cần được trigger. Ngoài ra nó cũng có thể phụ thuộc vào `Domain Service` - điều phối các domain process giữa các entities với nhau.

Ngoài ra sau khi thực thiện xong use case logic, Application service có thể sẽ thông báo cho toàn hệ thống biết nên do đó application service có thể sẽ phụ thuộc cả vào `dispatcher` để có thể trigger event.

Ở hình trên chúng ta sử dụng interface cho các persistence engine và các các repositories. Chúng có các mục đích như sau:

- Persistance interface là abstraction layer phía trên ORM để từ đó ta có thể dễ dàng thay đổi ORM mà không làm ảnh hưởng đến Application Core.
- Repository interface là abstraction của persistence engine. Giả sử ta muốn chuyển từ MySQL sang MongoDB. Persistence interface sẽ không thay đổi mà chỉ có class implement nó thay đổi mà thôi. Do đó dù là MySQL hay MongoDB, chỉ cần đáp ứng được mechanism của interface là được.

### Khi có Command/Query Bus

![4_3 UMLish_2](https://user-images.githubusercontent.com/15076665/224055490-1ea5c1e8-0e7d-4f94-b970-64eca4b827f1.png)

Lúc này controller sẽ phụ thuộc vào command và query. Command và query sẽ được khởi tạo và truyền vào Bus - Bus sẽ có nhiệm vụ tìm handler phù hợp đển xử lí các commands.

## Tổng kết

Mục tiêu cuối cùng của chúng ta vẫn là tạo ra một codebase có tính phụ thuộc thấp để có thể dễ dàng thay đổi, mở rộng.

Tất cả các thông tin phía trên chỉ mang tính chất khái niệm, tuy nhiên nếu hiểu được tất cả các khái niệm đó, chúng ta có thể xây dựng được một kiến trúc và một hệ thống "khoẻ mạnh".

Thế nhưng **phát triển ứng dụng là một lĩnh vực có tính thực tế cao, tuỳ vào từng trường hợp, chúng ta sẽ áp dụng những kiến thức của mình sao cho linh hoạt nhất, đó mới chính là những yếu tố cấu thành nên kiến trúc hệ thống**.

**Chúng ta cần hiểu các patterns thế nhưng chúng ta cũng cần phải nghĩ và thực sự hiểu rằng ứng dụng của chúng ta cần gì, và liệu rằng chúng ta có thể đi bao xa trong công cuộc chia tách các components**.

Quyết định này sẽ phụ thuộc vào rất nhiều yếu tố có thể là yêu cầu về chức năng của hệ thống cũng có thể là vòng đời của sản phẩm cũng như trình độ của team dev, ...

Trên đây là tất cả những gì tôi đã hệ thống hoá được.

Ngoài ra tôi cũng mở rộng thêm một vài ý tưởng ở bài viết [Hơn cả concentric layers](https://herbertograca.com/2018/07/07/more-than-concentric-layers/)

Vậy làm thế nào để có thể thực hiện được những yêu cầu trên cho code base của bạn ? Đó chính là chỉ đề trong bài viết tiếp theo của tôi: làm cách nào để áp dụng kiến trúc và domain vào code.
