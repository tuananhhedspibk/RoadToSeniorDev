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

