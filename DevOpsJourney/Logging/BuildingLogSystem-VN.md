# Thiết kế hệ thống logging

Với những bạn đã và đang lập trình thì việc "check log" trong quá trình debug không còn là một điều xa lạ. Nếu không có log, chúng ta không thể biết được file nào, dòng nào trong file đó đang phát sinh lỗi để có thể sửa chữa kịp thời.

Đó là đối với những chương trình hoặc file đơn lẻ, thế nhưng với những hệ thống phục vụ người dùng trong thực tế thì việc "check log" lại càng trở nên quan trọng hơn vì nó giúp chúng ta:

- Điều tra, phát hiện ra những lỗi đang xảy ra làm ảnh hưởng đến người dùng cuối.
- Theo dõi "sức khoẻ" của hệ thống để từ đó có thể phát hiện ra những "dấu hiệu bất thường" trước khi nó kịp phát tác làm ảnh hưởng đến hệ thống.
- ...

Qua đó chúng ta có thể thấy việc "check log" đóng vai trò rất quan trọng trong quá trình phát triển cũng như vận hành một phần mềm hoặc một hệ thống (còn gọi là monitoring), thế nên việc thiết kế và triển khai một hệ thống log tốt sẽ giúp quá trình monitoring trở nên dễ dàng hơn rất nhiều.

Trong bài viết lần này tôi xin phép được gửi đến bạn đọc những kiến thức thực tế mà tôi đã tiếp thu được về việc xây dựng một hệ thống logging, một phần là để bạn đọc cảm nhận rõ hơn về tầm quan trọng của log khi vận hành hệ thống, một phần là để bạn đọc có được cho mình một tài liệu tham khảo khi muốn xây dựng một hệ thống logging trong thực tế. Rất mong được bạn đọc đón nhận.

## Logging Policy

Dưới đây là những câu hỏi chúng ta cần đặt ra trước khi xây dựng một hệ thống log.

- **Why**: Log nhằm mục đích gì ?
- **Who**: Hệ thống hay module nào tạo ra log (API, APP, ...) ?
- **When**: Khi nào, thời điểm nào thì đưa ra log ?
- **Where**: Đưa ra log ở đâu (log sẽ được gửi đến Slack, BigQuery, ...) ?
- **What**: Log về cái gì ?
- **How**: Cách thức đưa ra log là như thế nào ?

## Log level

Sau khi đã hiểu rõ được mục đích của mình khi đưa ra log, chúng ta cần "phân cấp" log.

| Log level | Khái niệm                                                                                                           | Cách đối ứng                               | Ví dụ                                                               |
| --------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------- |
| FATAL     | Level này gây cản trở đến việc vận hành của hệ thống                                                                | Cần sửa ngay                               | Không thể kết nối tới DB                                            |
| ERROR     | Việc thực thi gây ra những lỗi ngoài dự tính                                                                        | Sửa trong thời gian hoạt động của hệ thống | Không thể gửi mail                                                  |
| WARN      | Không phải lỗi nhưng là những vấn đề như: input không như mong muốn hoặc thực thi không như mong muốn               | Refactor định kì                           | API xoá dữ liệu một cách định kì                                    |
| INFO      | Thông báo khi bắt đầu hoặc kết thúc một xử lí, transaction. Cũng có thể là việc đưa ra các thông tin cần thiết khác | Không cần phải sửa                         | Đưa ra nội dung của req / res, hoặc khi bắt đầu hoặc kết thúc batch |
| DEBUG     | Thông tin liên quan đến trạng thái hoạt động của hệ thống                                                           | Không đưa ra ở môi trường production       | Có thể đặt ở các hàm bên trong app                                  |
| TRACE     | Thông tin chi tiết ở mức độ cao hơn DEBUG                                                                           | Không đưa ra ở môi trường production       |                                                                     |

## Case

Sau khi đã rõ các cấp độ log, chúng ta cần làm rõ "các loại log" mà mình muốn đưa ra. Trong phần này đối với từng loại log tôi sẽ đi trả lời 6 câu hỏi:

- Why
- Who
- When
- Where
- What
- How

như đã nói chi tiết ở phần **Logging Policy** phía trên.

### System log

- Why: Những log này sẽ sử dụng để debug khi hệ thống có lỗi.
- Who: Bản thân hệ thống sẽ sinh ra log.
- When: Đưa ra log khi phát sinh lỗi.
- Where:
  - `FATAL / ERROR`: Thông báo đến một kênh mà dev có thể nhận ra và xử lí kịp thời.
  - `WARN / INFO`: Đưa ra bên trong hệ thống hoặc các tool quản lí log.
  - `DEBUG / TRACE`: Đưa ra ở dạng `console.log` trên môi trường `staging`.
- What:
  - `FATAL / ERROR`: Stacktrace.
  - `WARN / INFO / DEBUG/ TRACE`: Nội dung muốn thông báo.
- How:
  - `FATAL / ERROR`: Đưa ra log thông qua các tool quản lí log hoặc đưa tới Slack, SMS, ... (dưới hình thức `push`).
  - `WARN / INFO / DEBUG / TRACE`: Đưa ra log thông qua các tools quản lí log hoặc bên trong hệ thống, ... (dưới hình thức `pull`).

### Access log

- Why: Đưa ra log để theo dõi việc gửi, nhận request.
- Who: Bản thân hệ thống hoặc từ phía infrastructure.
- When: Đưa ra tại thời điểm access (gửi, nhận request)
- Where: Chủ yếu là log ở `level INFO`, nên sẽ đưa ra ở dạng pull. Ngoài ra do lượng log có thể nhiều nên tốc độ tìm kiếm log cũng là việc cần phải chú ý.
- What: Đưa ra **ai**, **bằng cách nào**, **khi nào** access đến hệ thống.
- How: Tuỳ vào đối tượng và mục tiêu mà có sự khác biệt.

### Action log

- Why: Đưa ra để tiến hành phân tích hành vi của người dùng, qua đó nhằm cải thiện service.
- Who: Bản thân hệ thống hoặc external tools.
- When: Khi phát sinh ra actions (hành động).
- Where: Các tool có khả năng phân tích log (BigQuery, BI, ...).
- What: Tuỳ vào mục tiêu.
- How: Tuỳ vào đối tượng và mục tiêu mà có sự khác biệt.

### Auth log

- Why: Đưa ra nhằm mục đích theo dõi quá trình xác thực người dùng.
- Who: Bản thân hệ thống.
- When: Khi tiến hành xác thực người dùng.
- Where: Chủ yếu là `level INFO`, dưới hình thức `pull`.
- What: Đưa ra **ai**, **bằng cách nào**, **khi nào** thì được xác thực.
- How: Tuỳ vào phương thức xác thực mà có sự khác biệt

## Ví dụ minh hoạ

Lí thuyết như vậy là đủ, giờ chúng ta hãy cùng nhau thử triển khai nó trong một project đơn giản. Bạn đọc có thể tham khảo source code tại: <https://github.com/tuananhhedspibk/NewAnigram-BE-DDD>

### Lựa chọn thư viện log

Ở đây tôi sẽ sử dụng thư viện `log4js` (<https://github.com/log4js-node/log4js-node>). Lí do rất đơn giản vì bản thân log4js cũng tổ chức hệ thống phân cấp log của mình giống như tư tưởng mà tôi đã trình bày trong phần `Log level` ở trên.

### Triển khai

#### Bước 1 - Định nghĩa class Logger

Đầu tiên tôi sẽ định nghĩa một class Logger riêng như sau:

```ts
class Logger {
  public default: log4js.Logger;
  public system: log4js.Logger;
  public api: log4js.Logger;
  public access_req: log4js.Logger;
  public access_res: log4js.Logger;
  public sql: log4js.Logger;
  public auth: log4js.Logger;

  public fatal: log4js.Logger;
  public error: log4js.Logger;
  public warn: log4js.Logger;
  public info: log4js.Logger;
  public debug: log4js.Logger;
  public trace: log4js.Logger;

  constructor() {
    log4js.configure(loggerConfig);

    this.system = log4js.getLogger('system');
    this.api = log4js.getLogger('api');
    this.access_req = log4js.getLogger('access_req');
    this.access_res = log4js.getLogger('access_res');
    this.sql = log4js.getLogger('sql');
    this.auth = log4js.getLogger('auth');

    this.fatal = log4js.getLogger('fatal');
    this.fatal.level = log4js.levels.FATAL;

    this.error = log4js.getLogger('error');
    this.error.level = log4js.levels.ERROR;

    this.warn = log4js.getLogger('warn');
    this.warn.level = log4js.levels.WARN;

    this.info = log4js.getLogger('info');
    this.info.level = log4js.levels.INFO;

    this.debug = log4js.getLogger('debug');
    this.debug.level = log4js.levels.DEBUG;

    this.trace = log4js.getLogger('trace');
    this.trace.level = log4js.levels.TRACE;
  }
}
```

Tôi định nghĩa bên trong class Logger các `Log Levels` như:

- fatal
- error
- debug
- ...

ngoài ra còn có các `Log Types` như:

- system
- api
- access_req
- access_res
- ...

#### Bước 2 - "Áp dụng" Logger vào project

Project lần này tôi sử dụng framework NestJS để triển khai. Tôi sẽ áp dụng Logger vào project của mình dưới hình thức một "Interceptor", cụ thể hơn về "Interceptor" trong NestJS bạn đọc có thể tham khảo bài viết dưới đây của tôi để biết thêm chi tiết:

<https://viblo.asia/p/cac-concepts-co-ban-trong-nestjs-aAY4qepK4Pw>

Lí do tôi chọn "Interceptor" là hình thức triển khai log là do "Interceptor" có thể "bao ngoài" `request stream` cũng như `response stream` đến và đi từ API nên sử dụng Interceptor là cách dễ dàng nhất để có thể bắt được `request log` và `response log`. Tôi định nghĩa `LoggerInterceptor class` như sau:

```ts
export class LoggerInterceptor implements NestInterceptor {
  intercept(
    context: ExecutionContext,
    next: CallHandler<any>
  ): Observable<any> | Promise<Observable<any>> {
    // intercept() method sẽ "bao ngoài" request/ response stream

    /*
     * Lấy request object từ context
     * Sau đó cho request object đi qua hàm hiển thị log là "requestLogger"
     */
    const request = context.switchToHttp().getRequest();
    requestLogger(request);

    /*
     * Lấy response object từ context
     * Sau đó cho response object đi qua hàm hiển thị log là "responseLogger" & "responseErrorLogger"
     */
    const response = context.switchToHttp().getResponse();

    return next.handle().pipe(
      // 200 - Success Response
      map((data) => {
        responseLogger({ requestId: request._id, response, data });
      }),
      // 4xx, 5xx - Error Response
      tap(null, (exception: HttpException | Error) => {
        try {
          responseErrorLogger({ requestId: request._id, exception });
        } catch (e) {
          logger.access_res.error(e);
        }
      })
    );
  }
}
```

bạn đọc có thể tham khảo chi tiết hơn tại đường dẫn sau: <https://github.com/tuananhhedspibk/restful-app/blob/main/libs/interceptor/logger/index.ts>

Tôi định nghĩa 3 methods:

- requestLogger
- responseLogger
- responseErrorLogger

lần lượt như sau:

```ts
export const requestLogger = (request: Request) => {
  const { ip, originalUrl, method, params, query, body, headers } = request;

  // logTemplate sẽ bao gồm: now(thời gian) ip method url request_object
  const logTemplate = '%s %s %s %s %s';
  const now = dayjs().format('YYYY-MM-DD HH:mm:ss.SSS');

  const logContent = util.formatWithOptions(
    { colors: true },
    logTemplate,
    now,
    ip,
    method,
    originalUrl,
    JSON.stringify({
      method,
      url: originalUrl,
      userAgent: headers['user-agent'],
      body: _maskFields(body, 'password'),
      params,
      query,
    })
  );

  // Gọi đến access_req logger object đã được định nghĩa trước đó
  logger.access_req.info(logContent);
};

// Hiển thị success response log
export const responseLogger = (input: {
  requestId: number;
  response: Response;
  data: any;
}) => {
  const { requestId, response, data } = input;

  const log: ResponseLog = {
    requestId,
    statusCode: response.statusCode,
    data,
  };

  // Gọi đến access_res logger object đã được định nghĩa trước đó
  logger.access_res.info(JSON.stringify(log));
};

// Hiển thị error response log
export const responseErrorLogger = (input: {
  requestId: number;
  exception: HttpException | Error;
}) => {
  const { requestId, exception } = input;

  const log: ResponseLog = {
    requestId,
    statusCode:
      exception instanceof HttpException ? exception.getStatus() : null,
    message: exception?.stack || exception?.message,
  };

  // Gọi đến access_res logger object đã được định nghĩa trước đó
  logger.access_res.info(JSON.stringify(log));
  logger.access_res.error(exception);
};
```

bạn đọc có thể tham khảo cụ thể hơn tại đường dẫn sau: <https://github.com/tuananhhedspibk/restful-app/blob/main/libs/middleware/logger/index.ts>

Sau khi đã định nghĩa xong "LoggerInterceptor" cũng các methods bổ trợ, tôi sẽ "apply" interceptor này vào main app của mình như sau:

```ts
const app = await NestFactory.create(AppModule);

app.useGlobalInterceptors(new LoggerInterceptor());
```

rất đơn giản vì đây là một tính năng sẵn có của framework NestJS, cụ thể hơn về source code bạn đọc có thể tham khảo tại đây: <https://github.com/tuananhhedspibk/restful-app/blob/main/src/main.ts>

Vậy thì với `fatal` hay `debug` log thì sao ? Các loại log này sẽ được tôi sử dụng trong tầng nghiệp vụ (usecase) hoặc tầng infrastructure với các mục đích:

- Thông báo các lỗi mang tính "chí mạng" như việc kết nối tới DB bị "ngắt".
- Có cơ sở để debug nếu user gặp lỗi.

Rất đơn giản thôi, chỉ cần làm như sau là được

```ts
logger.fatal.error('Error message');
```

Bạn đọc có thể tham khảo một ví dụ tại: <https://github.com/tuananhhedspibk/restful-app/blob/main/src/users/application/command/signup/handler.ts#L35>

Và đây là kết quả:

Đầu tiên là với `access request log` và `response log` (khi không xảy ra lỗi)

![Screenshot 2024-05-16 at 23 05 12](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/80eba6d1-bc8b-4dc0-ac90-ccb04da67840)

bạn đọc có thể thấy các thông tin liên quan đến request như `method`, `body`, .. cũng như thông tin liên quan đến response là `statusCode` đều được hiển thị rõ ràng.

Còn trong trường hợp xảy ra lỗi:

![Screenshot 2024-05-16 at 23 05 26](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/20c65d40-8941-495d-ae53-5bf92959ee0e)

thì loại lỗi, error message cũng đều được hiển thị.

Tiếp theo là `fatal log` sẽ như sau:

![Screenshot 2024-05-16 at 23 09 21](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/4e9a06ca-8c65-4f2a-970c-571981e78bf4)

error message và error stack cũng được show ra.

## Tổng kết

Vậy là trong bài viết lần này tôi đã chia sẻ được với bạn đọc cách thức thiết kế cũng như triển khai một hệ thống logging cơ bản. Ví dụ minh hoạ lần này không quá phức tạp nhưng tôi hi vọng rằng bạn đọc có thể hiểu được tầm quan trọng cũng như sự cần thiết của việc xây dựng một hệ thống log "chỉnh chu" sẽ giúp ích rất nhiều trong quá trình vận hành một hệ thống thực tế đặc biệt là khi hệ thống gặp lỗi hoặc trục trặc.

Cảm ơn bạn đọc đã đón nhận, xin hẹn gặp lại ở các bài viết tiếp theo. Happy coding.
