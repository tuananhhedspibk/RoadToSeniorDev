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

#### Retryable Exception

Đây là loại lỗi khá đơn giản, ta có thể giải quyết nó bằng cách "thử lại" thao tác thêm một lần nữa. Một ví dụ kinh điển đó là khi hệ thống của bạn tương tác với hệ thống bên ngoài và gặp một vài sự cố kiểu như:

- Timeout.
- Kết nối bị mất tạm thời.

Với những lỗi như vậy ta chỉ cần "thử lại" vài lần là đủ. Với lỗi dạng `Kết nối HTTP tới Data Store bị ngắt quãng`, ta thường sẽ xử lí bằng cách "thử lại - retry".

Retry là một công cụ phổ biến tuy nhiên hãy nhớ đến **retry limit**, bạn cũng có thể sử dụng **circuit breaker**, **exponential backoff**, ...

> Xin được nhấn mạnh lại là Retryable exception là một lỗi phổ biến mà không chỉ mỗi hệ thống hướng sự kiện mới gặp phải

#### Requeable Exception

Vấn đề ở đây đó là hệ thống của bạn không thể xử lí message hiện thời được (nguyên nhân có thể là do một vài services trong hệ thống đang gặp sự cố về tính toàn vẹn của dữ liệu hoặc quá trình xử lí dữ liệu của service khác đang bị trễ một vài phút), do đó tại một thời điểm nào đó trong tương lai khi các services của hệ thống ở trong trạng thái sẵn sàng 100% thì message sẽ được xử lí.

Mặt khác nếu bạn không tiến hành xử lí những lỗi của các services nêu trên thì việc xử lí các messages khác cũng sẽ bị ngưng trệ. **Distributed transactional systems** thường xuyên gặp những lỗi tương tự như vậy.

Vậy giải pháp ở đây là gì? Ta sẽ lấy message đó ra, đưa xuống cuối hàng đợi xử lí và "retry" tại một thời điểm khác.

Do đó với lỗi `processor không thể tìm thấy dữ liệu từ data warehouse nguyên nhân do một vài services khác bị trễ trong việc tìm kiếm dữ liệu trên` ta có thể xử lí bằng cách **Requeing**.

`Retryable Exception` & `Requeable Exception` có mối liên hệ với nhau. Cụ thể như sau, nếu trong trường hợp số lần retry đạt đến ngưỡng tối đa nhưng hệ thống vẫn không thể xử lí được message thì lúc này nguyên nhân chắc chắn **không phải** do sender, message hoàn toàn không có vấn đề, vấn đề nằm ở service của chúng ta, nếu cứ "retry một cách mù quáng" thì sẽ làm ảnh hưởng đến việc xử lí các messages khác, do đó

> Khi retry đạt tới ngưỡng tối đa, hãy nghĩ đến requeue

#### Poison Pill

Poison pill là một message mà phía consumer không thể xử lí được dù retry bao nhiêu lần đi chăng nữa. Ví dụ nếu message không thể `deserializing` và nó không bao giờ được xử lí, dẫn đến việc toàn bộ các messages phía sau bên trong queue cũng sẽ không được xử lí theo.

Dù bản thân queue cũng có cơ chế tự loại bỏ đi các message "lỗi" kiểu vậy, thế nhưng cho đến khi message này bị loại bỏ, nó sẽ làm cho toàn bộ hệ thống bị "kẹt", đây là một điều rất nguy hiểm.

#### Droppable Exception

Với ngoại lệ thuộc dạng này, ta có thể bỏ qua mà không xử lí vì bản thân message là không hợp lệ. Lấy ví dụ như sau: message sử dụng enum type không có trong định nghĩa cho trước, đây là lỗi được biết từ trước do đó ta có thể nhận về message nhưng không xử lí.

Với ngoại lệ thuộc dạng này chúng ta sẽ định danh như sau: `Message nhận được không được hỗ trợ xử lí bởi ứng dụng`.

#### DLQable Exception

Đây còn được gọi là `Unhandled Exceptions`. Đây là những lỗi mà bạn không thể đoán được trước, nhưng cũng cần phải tìm hiểu nguyên nhân chính gây ra nó và đồng thời không thể để nó trở thành `Poison Pill`.

Nên ở đây ta sẽ đưa nó vào một queue khác đó là `Dead Letter Queue` - để sau đó ta có thể phân tích, tìm ra nguyên nhân gây ra lỗi cũng như các ngoại lệ khác phát sinh đi kèm theo message này.

Ngoài việc phân tích message, ta cũng có thể điều chỉnh lại nó để message trở nên hợp lệ.

Định danh cho ngoại lệ này chính là `Consumer không thể xử lí message vì cấu trúc JSON không hợp lệ`.

## Code

Dưới đây sẽ là code minh hoạ cho những ý tưởng nói ở trên.

1. Core error-handling logic sẽ nằm ở trong `consumer` vì flow bắt đầu từ đây. Ở đây bạn sẽ thấy được các loại exception khác nhau được xử lí như thế nào cũng như cái cách mà `Retryable Exception` biến thành `Requeable Exception`

```java
public void consume(Message<String> message) {
  try {
    final EventHandler handler = subscriptionMap.get(event);
    processWithRetry(handler, message);
  } catch (RequeableException e) {
    failureHandler.reQueue(message, e);
  } catch (DroppableException e) {
    failureHandler.dropMessage(message, e);
  } catch (Exception e) {
    failureHandler.putOnDlq(message, e);
  }
}

private void processWithRetry(EventHandler handler, Message<String> message) throws Exception {
  retryTemplate.execute(context -> {
    handler.handle(message); // Processing delegated to the handler method
    return null;
  }, context -> {
    final Exception originalError = (Exception) context.getLastThrowable();
    if (originalError instanceof RetryableException) {
      throw new RequeableException(originalError); // After cutoff RetryableException exception becomes RequeableException
    }
  }
}
```

2. Retry config

```java
 public RetryTemplate eventConsumerRetryTemplate(int maxAttempts) {
  return RetryTemplate.builder()
    .retryOn(RetryableException.class)
    .maxAttempts(maxAttempts)
    .exponentialBackoff(100, 2, 300)
    .traversingCauses()
    .build();
}
```

3. Handler/processor xử lí message, giới hạn lại method thực sự đưa ra exception, đóng gói các exceptions lại vào trong 4 kiểu mà chúng ta đã định nghĩa

```java
public void handle(Message<String> message)
  throws RetryableException, DroppableException, DlqableException {
  try {
    doStuff(message);
  } catch (IllegalArgumentException ex) {
    throw new DlqableException(ex);
  }
}
```

Các đoạn code phía trên mang tính tổng quát khá cao, tuy nhiên chúng có thể giúp bạn hiểu được ý tưởng cốt lõi ở đây đó là **Chúng ta có thể xử lí lỗi theo 4 cách thông qua việc phân loại các lỗi đó thành 4 loại**, bằng cách làm này, chúng ta có thể đưa ra một cơ chế xử lí lỗi rõ ràng cho hệ thống và giúp hệ thống có thể "dễ đoán định" hơn.

