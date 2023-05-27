# Các concepts cơ bản trong Nestjs

Bài viết được lược dịch từ [nguồn](https://zenn.dev/morinokami/articles/nestjs-overview)

## Tổng quan

Theo như document của nestjs ta có 8 khái niệm cơ bản như sau:

- Controllers
- Providers
- Modules
- Middleware
- Exception Filters
- Pipes
- Guards
- Interceptors

3 khái niệm đầu tiên Controllers, Providers, Modules sẽ lần lượt đảm nhiệm các nhiệm vụ liên quan đến routing các request từ client cũng như business logic.

5 khái niệm còn lại đều liên quan đến đường đi của request cũng như response và được minh hoạ như trong sơ đồ dưới đây

![zs4f5nwd0mf2m95sdpcye34x1gc8](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/2b9fe624-f46e-4a25-8795-89005a47764e)

Các đường chấm màu ghi sẽ là HTTP request, HTTP response và Exception (trong thực tế thì Exception cũng sẽ được trả về dưới hình thức HTTP response). App Module sẽ là root module chứa các modules con ở phía trong nó, trong mỗi module con sẽ lại có các controller và service.

Trên đường đi của Request sẽ là:

- Middleware
- Guard
- Interceptor
- Pipe

Còn với Response sẽ là:

- Interceptor
- Exception Filter (với trường hợp của Exception)

Khi đăng kí (Exception filter, Pipe, Guard, Interceptor) với app, ta sẽ lần lượt có 4 levels như sau:

- Global
- Controller
- Method
- Param

## Providers

Sẽ được khai báo kèm theo `@Injectable()` với mục đích dùng cho DI. Thông thường đây sẽ là nới thực hiện các task được uỷ nhiệm từ phía controller.

## Middleware

Là hàm được gọi phía trước route, có khả năng truy cập đến `Request Object`, `Response Object`. Có khả năng đảm nhận những công việc sau:

- Thay đổi, chỉnh sửa `Request Object`, `Response Object`.
- Kết thúc vòng đời của Request hoặc Response.

Cũng được khai báo thêm với `Injectable`

```ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

// classとして定義する
@Injectable() // @Injectable() デコレータの適用
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...'); // Middleware の処理
    next(); // 次の関数へとコントロールを引き渡す
  }
}

// 関数として定義する
export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
}
```

Có 2 cách sử dụng Middleware:

① Sử dụng theo từng module, controller cụ thể.

```ts
export class AppModule implements NestModule { // NestModule インターフェースの実装
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware) // Middleware の適用
      .forRoutes(CatsController); // 適用対象の Route を指定
  }
}
```

② Sử dụng cho global module.

```ts
const app = await NestFactory.create(AppModule);
app.use(LoggerMiddleware);
```

## Exception filters

Thường được định nghĩa với decorator là `Catch()`. Được sử dụng để "bắt" các ngoại lệ không được xử lí, nó sẽ kiểm soát các ngoại lệ (HttpException) khi ngoại lệ được gửi về phía client.

Cơ chế mặc định ở đây đó là nó sẽ tìm ra ngoại lệ, sau đó sẽ chuyển sang dạng HTTP Response.

```ts
@Catch(HttpException) // @Catch() デコレータの適用、HttpException をハンドルすることを宣言
export class HttpExceptionFilter implements ExceptionFilter { // ExceptionFilter インターフェースの実装
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    // レスポンスを加工
    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

Exception filter có thể được sử dụng cho cả 3 level: method, controller, global với cách thức như sau:

① Method

```ts
@Post()
@UseFilters(HttpExceptionFilter) // Exception filter を登録
async create(@Body() createCatDto: CreateCatDto) {
  // ...
}
```

② Controller

```ts
@UseFilters(HttpExceptionFilter)
export class CatsController {}
```

③ Global

```ts
const app = await NestFactory.create(AppModule);
app.useGlobalFilters(HttpExceptionFilter);
```

## Pipes
