# Event-Sourcing là gì ?

Bài viết được lược dịch từ: https://levelup.gitconnected.com/basics-of-event-sourcing-12ebe0b86788

## Tổng quan

Event-Sourcing là cơ chế lưu trữ dữ liệu thường được sử dụng trong DDD và CQRS. Các khái niệm này là độc lập với nhau nhưng chúng lại bổ sung cho nhau một cách hoàn hảo.

Việc lưu dữ liệu thường được thực hiện bởi thao tác **CRUD**. Ở đây 4 **thao tác với database** đó là (CREATE, READ, UPDATE, DELETE). Trong đó thì 2 thao tác UPDATE và DELETE là không cùng loại với 2 thao tác kia vì bản chất của chúng liên quan tới "destructive operation - thao tác huỷ", với UPDATE ta sẽ "bỏ đi" dữ liệu cũ và "thay thế" bằng dữ liệu mới.

Do đó tư tưởng của Event-Sourcing đó là **từ bỏ UPDATE và DELETE** để từ đó **chỉ sử dụng CREATE và READ** mà thôi.

Với event-sourcing, mọi sự thay đổi đều được coi là một **new entry** được thêm mới vào list (append-only log), các entries này sẽ **không bao giờ thay đổi** cũng như **không bao giờ bị loại bỏ**.

Do đó database trong event-sourcing thường được gọi là **event-store**, các events sau đó sẽ được **reload** và **aggregate** để tái hiện lại dữ liệu (có thể là ở thời điểm hiện tại và cũng có thể là ở một thời điểm nào đó trong quá khứ).

## Nhược điểm của Event-Sourcing

Do các event sẽ được "append" vào event-store nên do đó event-store sẽ ngày càng phình to, từ đó sẽ làm cho quá trình aggregate để tái hiện lại dữ liệu sẽ lâu hơn.

Vấn đề có quá nhiều event records có thể được giải quyết bằng snapshot

## CRUD vs Event-Sourcing

CRUD cũng có những ưu điểm riêng như hiệu năng cao khi lưu trữ raw data, không yêu cầu logic đặc biệt.

Thế nhưng một nhược điểm thấy rõ của CRUD đó là tình trạng bị conflict dữ liệu khi các transaction có thể "đè" lên nhau, còn Event-Sourcing sẽ ít khi xảy ra tình trạng conflict dữ liệu này, nguyên nhân là bởi các event entries sẽ chỉ lưu dữ liệu ở dạng "delta" mà thôi.

## Sử dụng database nào cho Event-Sourcing ?

Do Event-Sourcing chỉ cần "INSERT" và "SELECT", không cần những câu JOIN quá phức tạp cũng như CTE (common table expression) nên do đó những yêu cầu về DB cho EventSourcing cũng khá thấp.

Việc lưu trữ các events kiểu này dưới dạng BLOB hoặc JSON sẽ đem lại nhiều ưu việt hơn nên do đó NoSQL DB (MongoDB) là một sự lựa chọn tốt khi nó không cần schema cũng như tổ chức dữ liệu dưới format JSON.

Ngoài ra **Apache Kafka** cũng sẽ là một sự lựa chọn cho event-store thế nhưng bạn chỉ nên sử dụng Apache Kafka khi đã hiểu rõ về tình huống của mình.

## DDD và Event-Souring

Trong DDD cũng có một khái niệm đó là "domain events" - khái niệm này và khái niệm event trong event-sourcing đều mô tả các **sự kiện liên quan đến business logic của hệ thống**

> Do domain-events thường là kết quả của các commands nên các domain events sẽ được lưu trữ trong event-store

## Tổng kết

Event-Sourcing có rất nhiều ưu điểm cho việc lưu trữ dữ liệu thế nhưng với những dữ liệu cần phải có form chuẩn hoặc phải chú ý đến **duplicate** hoặc **uniqueness** thì event-sourcing lại không phải là một sự lựa chọn tốt.

Do đó event-sourcing không phù hợp với CRUD và ngược lại, thế nhưng tuỳ vào từng tình huống mà chúng lại có thể bổ sung cho nhau, do đó hãy quyết định việc lựa chọn công cụ như thế nào tuỳ theo tình huống mà bạn gặp phải nhé.

Cảm ơn bạn đã đọc bài viết về Event-Sourcing của tôi, hẹn gặp lại ở các bài viết tiếp theo, xin cảm ơn.
