# Xử lí lỗi trong hệ thống hướng sự kiện

*Bài viết được lược dịch từ [nguồn](https://levelup.gitconnected.com/error-handling-in-event-driven-systems-1f0a7ef2cfb7)*

Xử lí lỗi là một phần quan trọng trong quá trình xây dựng một hệ thống. Xử lí lỗi tốt sẽ tăng **tính sẵn có** cho hệ thống, giúp cho hệ thống của chúng ta **dễ dàng dự đoán hơn** cũng như bảo vệ hệ thống khỏi trạng thái không nhất quán.

Với các REST API-based services, việc xử lí lỗi đã được hệ thống hoá khá đầy đủ với các **HTTP response code** cho các lỗi khác nhau có thể xảy ra trong hệ thống. Đồng thời kiến trúc dựa trên `REST` cũng được sử dụng rộng rãi hơn so với kiến trúc hướng sự kiện.

Trong bài viết lần này, tôi sẽ đưa ra một vài khía cạnh của việc xử lí lỗi với các hệ thống hướng sự kiện. Chúng ta sẽ thử nghiệm với một hệ thống, đưa ra điểm nghẽn, định danh chúng và rút ra các patterns nếu có thể.

## Tại sao việc xử lí lỗi lại quan trọng?

**REST** sử dụng cơ chế **tương tác hai chiều**, do đó bạn có thể "chuyển" các lỗi đến cho caller/ client để phía client tự xử lí lỗi nếu trong trường hợp bạn quên xử lí lỗi ở server, trong trường hợp này lỗi sẽ được client hiểu với định danh **500/ Internal Server Exception** và vẫn luôn có cơ hội để xử lí những lỗi đó.

Còn với hệ thống hướng sự kiện thì điều đó là không thể, nguyên nhân là do trong hệ thống hướng sự kiện thì dữ liệu sẽ đi theo một chiều duy nhất từ `producer` đến `consumer`. Ta vẫn có thể "truyền ngược" lại từ `consumer` đến `producer` được, thể nhưng đây không phải là giải pháp tối ưu trong mọi trường hợp. Hơn nữa nếu một event trong hệ thống hướng sự kiện bị lỗi sẽ dẫn đến **poison pill** và việc xử lí các messages còn lại sẽ bị ngưng lại hoàn toàn. Do đó việc xử lí lỗi trong hệ thống hướng sự kiện vô cùng quan trọng và phức tạp. Giờ chúng ta sẽ cùng nhau "làm giảm" sự phức tạp của nó

## Hệ thống

Giả sử ta có một hệ thống hướng sự kiện đơn giản như dưới đây:

![Screen Shot 2023-02-05 at 18 46 13](https://user-images.githubusercontent.com/15076665/216812082-f7021766-f24e-4e49-8b38-9b99317c081f.png)

1. `Event source` sẽ tạo ra các events/ messages cho ứng dụng.
2. Các messages/ events này sẽ được `Message queue` chuyền tới ứng dụng.
3. Ứng dụng gồm 3 phần: `Consumer`, `Processor`, `Writer`. Consumer sau khi nhận event/ message sẽ truyền tới cho processor, processor sẽ truy xuất dữ liệu từ Data Warehouse, sau đó truyền toàn bộ payload tới writer để writer ghi lên cache.

### Những điểm nghẽn tiềm tàng

Dưới đây là một vài lỗi mà chúng ta cần phải xử lí:

1. Consumer không thể hiểu được message vì nó gặp lỗi `cấu trúc JSON`.
2. Message nhận được không được hỗ trợ xử lí bởi ứng dụng.
3. Processor không thể tìm thấy dữ liệu cho message từ data warehouse.
4. Kết nối HTTP tới data warehouse bị mất
5. Hệ thống cache bị "sập" và không thể nhận requests tới được.

...

Nếu bạn đã từng làm việc với một hệ thống phân tán trước đó, bạn có thể thấy được rất nhiều vấn đề khác có thể phát sinh, tuy nhiên do phạm vi của bài viết nên tôi chỉ giới hạn một số lỗi tiêu biểu mà thôi.

### Phân loại lỗi

Sau đây chúng ta sẽ phân loại các lỗi có thể xảy ra cũng như cách xử lí chúng.

> Một trong những nguyên tắc tôi luôn tuân thủ đó là nghĩ về caller trước khi đưa ra exception. Ý tôi ở đây là nghĩ về những gì caller cần để có thể xử lí ngoại lệ cũng như caller sẽ làm gì sau khi nhận được ngoại lệ.

Có 4 loại ngoại lệ chính ở đây:

1. **Retryable Exception**
2. **Requeable Exception**
3. **Droppable Exception**
4. **DLQable Exception**

