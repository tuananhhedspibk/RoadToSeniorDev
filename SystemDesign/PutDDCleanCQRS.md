# Làm cách nào để kết hợp DDD, Hexagonal, Onion, Clean, CQRS, ...

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
