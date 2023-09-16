# Event Sourcing là gì ?

Bài viết được lược dịch từ: https://levelup.gitconnected.com/basics-of-event-sourcing-12ebe0b86788

## Tổng quan

Event sourcing là cơ chế lưu trữ dữ liệu thường được sử dụng trong DDD và CQRS. Các khái niệm này là độc lập với nhau nhưng chúng lại bổ sung cho nhau một cách hoàn hảo.

Việc lưu dữ liệu thường được thực hiện bởi thao tác **CRUD**. Ở đây 4 **thao tác với database** đó là (CREATE, READ, UPDATE, DELETE). Trong đó thì 2 thao tác UPDATE và DELETE là không cùng loại với 2 thao tác kia vì bản chất của chúng liên quan tới "destructive operation - thao tác huỷ", với UPDATE ta sẽ "bỏ đi" dữ liệu cũ và "thay thế" bằng dữ liệu mới.

Do đó tư tưởng của event sourcing đó là **từ bỏ UPDATE và DELETE** để từ đó **chỉ sử dụng CREATE và READ** mà thôi.

Với event-sourcing, mọi sự thay đổi đều được coi là một **new entry** được thêm mới vào list (append-only log), các entries này sẽ **không bao giờ thay đổi** cũng như **không bao giờ bị loại bỏ**.

Do đó database trong event-sourcing thường được gọi là **event-store**, các events sau đó sẽ được **reload** và **aggregate** để tái hiện lại dữ liệu (có thể là ở thời điểm hiện tại và cũng có thể là ở một thời điểm nào đó trong quá khứ).

## Nhược điểm của Event-Sourcing

Do các event sẽ được "append" vào event-store nên do đó event-store sẽ ngày càng phình to, từ đó sẽ làm cho quá trình aggregate để tái hiện lại dữ liệu sẽ lâu hơn.
