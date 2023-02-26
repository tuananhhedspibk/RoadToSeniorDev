# Xây dựng hệ thống xử lí lỗi bằng NodeJS

Dịch từ [nguồn](https://www.toptal.com/nodejs/node-js-error-handling)

> Xử lí lỗi không chỉ đơn thuần là giảm thời gian tìm bug cho dev mà còn là xây dựng một codebase với quy mô tương xứng với hệ thống của bạn

## Các loại lỗi trong NodeJS

Trong Node.js có 2 loại errors chính:

- **Operational errors**: runtime error, một số ví dụ: "Out of memory", "An invalid input for an API endpoint"
- **Programmer errors**: unexpected bugs, bản thân code có những vấn đề cần phải giải quyết. Ví dụ tiêu biểu: đọc property của "undefined" object. Các bug này thường do dev tạo nên chứ không liên quan đến operation.

## Xử lí lỗi

Với các lỗi đã được định nghĩa sẵn bởi Node.js, ta sẽ dễ dàng theo dõi thông tin xung quanh nó nhờ `Stacktrace` → từ đó ta có thể tìm ra nguyên nhân gốc rễ của lỗi.

Ngoài ra việc extends từ `Error class` cũng như bổ sung các thuộc tính khác như `HTTP status code` cũng sẽ giúp cho thông tin về lỗi trở nên chi tiết hơn

```TS
class BaseError extends Error {
  public readonly name: string;
  public readonly httpCode: HttpStatusCode;
  public readonly isOperational: boolean;

  constructor(
    name: string,
    httpCode: HttpStatusCode,
    description: string,
    isOperational: boolean,
  ) {
    super(description);
    Object.setPrototypeOf(this, new.target.prototype);

    this.name = name;
    this.httpCode = httpCode;
    this.isOperational = isOperational;

    Error.captureStackTrace(this);
  }
}

// free to extend from BaseError
class APIError extends BaseError {
  constructor(name, httpCode = HttpStatusCode.INTERNAL_SERVER, isOperational = true, description = 'Internal Server Error') {
    super(name, httpCode, isOperational, descritpion);
  }
}
```

Một vài `httpStatusCode` cơ bản có thể thêm ở đây:

```TS
export enum HttpStatusCode {
  OK = 200,
  BAD_REQUEST = 400,
  NOT_FOUND = 404,
  INTERNAL_SERVER = 500,
}
```

Cách sử dụng như sau:

```TS
const user = await User.getById(1);

if (!user) {
  throw new APIError(
    'NOT FOUND',
    HttpStatusCode.NOT_FOUND,
    true,
    'detailed explanation'
  );
}
```

## Xử lí lỗi bằng NodeJS một cách tập trung

Việc xây dụng một component với chức năng để xử lí lỗi sẽ giúp giảm thiểu đi việc trùng lặp code xử lí lỗi trong project. Component này chịu trách nhiệm cho việc **giúp cho lỗi bắt được trở nên dễ hiểu hơn** ví dụ như:

- Gửi thông báo đến system admin
- Chuyển event error đến monitoring service như Sentry.io và log chúng ra

![File_000](https://user-images.githubusercontent.com/15076665/179694787-49e0f4fc-fcbd-46e7-a2b5-721d2152da90.png)

Trước khi được gửi đến error-handling centralized thì lỗi sẽ được gửi đến error-handling middleware để tiến hành `phân biệt giữa các error types`.

```TS
try {
  userSerivce.addNewUser(req.body).then((newUser: User) => {
    res.status(200).json(newUser);
  }).catch ((error: Error) => {
    next(error);
  });
}
```

Và Error-handling centralized sẽ trông như sau:

```TS
export class ErrorHandler {
  public async handleError(err: Error): Promise<void> {
    await logger.error(
      'Error message from the centralized error-handling component',
      err,
    );
    await sendToSlack();
    await sendEventsToSentry();
  }

  public isTrustedError(error: Error) {
    if (error instanceOf BaseError) {
      return error.isOperational;
    }
    return false;
  }
}
```

Để dev có thể theo dõi bug một cách dễ dàng hơn, hãy tiến hành log error ra theo một format dễ nhìn nhất.

Một vài logger formatter tiêu biểu như:

- <https://github.com/expressjs/morgan>
- <https://github.com/winstonjs/winston>

Hai thư viện này sẽ giúp cung cấp log ở các format level khác nhau tuỳ theo level của error.

Với các `Programmer errors`, cách giải quyết tốt nhất đó là
B1. Crash app ngay lập tức
B2. Restart lại app với các tool như pm2

Nguyên nhân là bởi `Programmer errors` thường sẽ làm cho app kết thúc với một state không như mong muốn.

```TS
process.on('uncaughtException', (error: Error) => {
  errorHandler.handleError(error);
  if (!errorHandler.isTrustedError(error)) {
    process.exit(1);
  }
});
```

Với promies rejection ta có thể làm như sau:

```TS
// get the unhandled rejection and throw it to another fallback handler we already have.
process.on('unhandledRejection', (reason: Error, promise: Promise<any>) => {
  throw reason;
});

process.on('uncaughtException', (error: Error) => {
  errorHandler.handleError(error);
  if (!errorHandler.isTrustedError(error)) {
    process.exit(1);
  }
});
```
