# 9 Logging best practices

Bài viết được dịch từ [nguồn](https://medium.com/gitconnected/9-logging-best-practices-da9457e33305)

Logging là một công cụ hữu hiệu trong việc debug cũng như monitoring error.

Thế nhưng nội dung log cũng rất quan trọng, nếu log ra những thông tin không hữu ích thì chỉ tốn bộ nhớ cũng như chi phí duy trì log mà thôi.

Theo ý kiến của cá nhân tôi, logging nên được nhìn nhận với tư cách là "công dân hạng một" trong hệ thống. Đề từ đó, ta sẽ coi log như một "chức năng" trong hệ thống, áp dụng những best practice cho nó để đạt được mục đích của chúng ta.

Khi bắt đầu với logging, tôi khuyên rằng bạn KHÔNG NÊN sử dụng các thư việc logging của ngôn ngữ lập trình mà bạn đang sử dụng. Chúng thường thiếu đi các tính năng mà một logging system cần phải có và mục đích chính của các thư viện này chỉ là debug dưới môi trường local mà thôi.

Dưới đây là một vài thư viện log tiêu biểu:

- [Winston](https://github.com/winstonjs/winston) của NodeJS.
- [Log4J](https://logging.apache.org/log4j/2.x/) của Java.
- [Zap](https://pkg.go.dev/go.uber.org/zap) của Go.

## Best practices

### 1. Kiểm tra logs của bạn

Ý tôi ở đây không phải là unit test, mà là sau khi viết logs bạn nên xem xem:

- Nội dung log có đúng như kì vọng hay không ?
- Có dễ hiểu hay không ?
- Có thiếu sót gì hay không ?
- Liệu có cần bổ sung thêm thông tin hay ngữ cảnh (context) gì nữa hay không ?

Hãy chia sẻ với các thành viên trong team của bạn, hỏi họ xem họ có hiểu nội dung của log hay không ?

### 2. Đừng log ra những thông tin nhạy cảm

Đừng bao giờ log ra những thông tin nhạy cảm như `API keys`, `Password`, ... Việc log ra những thông tin như vậy sẽ làm tăng nguy cơ về bảo mật cho hệ thống.

```TS
// Wrong
logger.error("Unable to login", request);

// Correct
logger.error("Unable to login", {
  userName: request.user,
  password: request.pass ? "[HIDDEN]" : null,
  // additonal information4
});
```

### 3. Hãy đưa ra những message cụ thể

Logging chỉ có giá trị nếu message cung cấp các thông tin hữu ích, do đó hãy cung cấp một message với nội dung cụ thể, chi tiết.

```TS
// Wrong
logger.info("We're starting !");
// Task processing logic
logger.info("Task completed !");

// Correct
logger.info("Starting task", {
  name: taskName,
  params: params,
  startTime: startTime,
});
// Task processing logic
logger.info("Task Completed", {
  response: output.response,
  event: {
    action: taskName,
    duration: currentTime - startTime,
  },
});
```

### 4. Đừng log ra những messages quá cồng kềnh

Logging cũng tốn chi phí để duy trì, thông thường log sẽ được lưu trong một file và sau đó sẽ được upload lên một storage để từ đó nó được phân tích. Nếu hệ thống của bạn đáp ứng cả triệu requests thì đồng nghĩa với đó là việc sẽ có cả triệu triệu log được đưa ra và lưu vào trong file.

Do đó chỉ nên log ra "vừa đủ" những thông tin hữu ích để tiết kiệm chi phí bảo trì log nhất có thể.

### 5. Log ra mọi lỗi

Log là một công cụ hỗ trợ rất đắc lực cho dev trong quá trình debug, nếu bạn đang cần phải viết các đoạn code đảm nhận chức năng xử lí lỗi, hãy đảm bảo rằng bạn luôn luôn log ra lỗi trước khi đưa ra một lỗi khác cho phía user. Ví dụ:

```TS
try {

} catch (err) {
  logger.error('An errror occured', { error: err, args: args });
  throw new SystemError('Unable to procress request');
}
```

Nếu chúng ta không log ra lỗi thì sau này sẽ rất khó để hiểu tại sao ở đây ta lại "ném" ra `SystemError`.

### 6. Tận dụng các tính năng sẵn có của logger

Các thư viện logger đều hỗ trợ rất nhiều tính năng hữu ích, ví dụ như `Log Level`. Việc đưa ra log level giúp ta dễ dàng phân biệt được:

- Loại lỗi.
- Ngữ cảnh xảy ra lỗi.

Với môi trường production ta chỉ đưa ra `Info`, `Error` log, còn với môi trường dev ta sẽ đưa ra mọi loại log. Và đồng thời sử dụng log với đúng mục đích, đừng dùng `Info` log cho lỗi.

### 7. Đừng sử dụng debug level cho system monitoring data

Với việc theo dõi hệ thống (system monitoring) ta chỉ cần các thông tin ở múc độ **Info** & **Error** là đủ, chứ không cần các thông tin ở mức độ **Debug**.

### 8. Đảm bảo rằng ta lưu trace Ids trong log

Việc thêm trace Ids thường dùng cho các hệ thống phân tán. Hãy log chúng ra, việc debug trong một hệ thống phân tán mà không có trace Ids là một việc vô cùng vất vả.

### 9. Thiết lập các quy chuẩn tối thiểu cho hệ thống

Thiết lập một số lượng nhất định các trường dữ liệu cần phải log ra bên trong hệ thống của bạn. Nó sẽ bao gồm:

- Độ trễ.
- Khoảng thời gian request.
- Trace Ids.

Việc có một quy chuẩn như vậy sẽ giúp cho việc phát hiện ra các lỗi tiềm tàng về hiệu năng của hệ thống trở nên dễ dàng hơn. Chúng ta từ đó có thể "chủ động" hơn trong việc nắm bắt các lỗi có trong hệ thống.
