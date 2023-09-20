# Microservices kết với CQRS và Event Sourcing bằng TypeScript và NestJS

※ Bài viết được dịch từ: <https://medium.com/gitconnected/microservices-with-cqrs-in-typescript-and-nestjs-5a8af0a56c3a>

## Lí thuyết về DDD, CQRS và Event Sourcing

Như chúng ta đã biết, mọi thứ trong CQRS đều **khởi nguồn từ command**.

> Command sẽ đem lại sự thay đổi cho hệ thống

Command trong DDD có thể coi như **trigger cho một aggregate** để từ đó tạo ra một sự thay đổi. Trong sự tương tác giữa CQRS và DDD, CQRS có tích hợp **queue** (trong trường hợp này, sẽ là **command queue** hoặc **command bus**). Chúng ta sẽ:

1. Tạo một command.
2. "Đưa" command vào **command queue**.
3. Trả ra kết quả là một **aggregate**.

![Screen Shot 2023-09-20 at 22 37 09](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/assets/15076665/7cd0b0b9-44de-400f-aaa8-ece3cb338938)

Để có thể xử lí command, aggregate cần có **internal state**, với event sourcing, state sẽ được **restore lại từ data trong quá khứ**.

Command sau đó sẽ "sinh ra" các **domain events mới**, các events này sẽ được đưa vào **event store**

Kết quả thu được từ quá trình thực thi command sẽ được lưu và phản ánh vào **read/ view database** - dữ liệu này sẽ được truy cập sau đó thông qua queries.

Các domain events được sinh ra trong quá trình thực thi command sẽ được lưu vào trong **write database**, nhưng kết quả của các events đó sẽ được lưu vào **view/ read database**

Lấy ví dụ, khi ta thực hiện "TransferFundCommand" để chuyển khoản sang một tài khoản khác, "FundsTransferredEvent" được sinh ra và lưu vào tron event-store như hình minh hoạ dưới đây

![Screen Shot 2023-09-20 at 22 55 57](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/assets/15076665/0a2d0729-6dc4-4545-aff8-112ff0a091d3)
