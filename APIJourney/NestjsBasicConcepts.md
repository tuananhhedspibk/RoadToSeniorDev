# Các concepts cơ bản trong Nestjs

Bài viết được lược dịch từ:

- <https://zenn.dev/morinokami/articles/nestjs-overview>
- <https://medium.com/aws-tip/understanding-nestjs-architecture-f257d054211d>

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

Sẽ được khai báo kèm theo `@Injectable()` với mục đích dùng cho DI. Thông thường đây sẽ là nơi thực hiện các task được uỷ nhiệm từ phía controller.

## Middleware

Là hàm được gọi phía trước route, có khả năng truy cập đến `Request Object`, `Response Object`. Có khả năng đảm nhận những công việc sau:

- Thay đổi, chỉnh sửa `Request Object`, `Response Object`.
- Kết thúc vòng đời của Request hoặc Response.

Cũng được khai báo thêm với `Injectable`

```ts
import {Injectable, NestMiddleware} from "@nestjs/common";
import {Request, Response, NextFunction} from "express";

// classとして定義する
@Injectable() // @Injectable() デコレータの適用
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log("Request..."); // Middleware の処理
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
export class AppModule implements NestModule {
  // NestModule インターフェースの実装
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
export class HttpExceptionFilter implements ExceptionFilter {
  // ExceptionFilter インターフェースの実装
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    // レスポンスを加工
    response.status(status).json({
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

Được định nghĩa đi kèm với `@Injectable()`, và implement `PipeTransform` interface. Mục đích chính của việc sử dụng pipe đó là:

- Thay đổi dữ liệu đầu vào (format dữ liệu đầu vào)
- Validation: nếu dữ liệu không vấn đề gì thì ta sẽ tiến hành xử lí tiếp theo còn nếu không thì sẽ đưa ra ngoại lệ.

Có tất cả 9 loại pipes:

- ValidationPipe
- ParseIntPipe
- ParseFloatPipe
- ParseBoolPipe
- ParseArrayPipe
- ParseUUIDPipe
- ParseEnumPipe
- DefaultValuePipe
- ParseFilePipe

Ví dụ:

```ts
@Injectable() // @Injectable() デコレータの適用
export class ParseIntPipe implements PipeTransform<string, number> {
  // PipeTransform インターフェースの実装
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10); // データの変換
    if (isNaN(val)) {
      throw new BadRequestException("Validation failed"); // Pipe を適用できないケースは例外を送出
    }
    return val;
  }
}
```

Pipe được sử dụng ở cả 4 levels: param, method, controller, global.

```ts
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id) { // パラメータ id に対する Pipe を登録
  return this.catsService.findOne(id);
}
```

```ts
@Post()
@UsePipes(ValidationPipe) // Pipe を登録
async create(@Body() createCatDto: CreateCatDto) {
  // ...
}
```

```ts
const app = await NestFactory.create(AppModule);
app.useGlobalPipes(ValidationPipe);
```

## Guards

Được định nghĩa với `@Injectable()`, implements `CanActivate` interface

Thường có nhiệm vụ quyết định xem có nên xử lí request không dựa theo (quyền, role, ACL, ...)

```ts
@Injectable() // @Injectable() デコレータの適用
export class AuthGuard implements CanActivate {
  // CanActivate インターフェースの実装
  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest<Request>();
    return validateRequest(request); // リクエストに対する何らかの検証 (true であれば次の処理へと進む)
  }
}
```

Guard có thể được sử dụng ở method, controller, global levels.

```ts
@Post()
@UseGuards(AuthGuard) // Guard を登録
async create(@Body() createCatDto: CreateCatDto) {
  // ...
}
```

```ts
@UseGuards(AuthGuard)
export class CatsController {}
```

```ts
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(AuthGuard);
```

## Interceptors

Được định nghĩa với `@Injectable()`, implements `NestInterceptor` interface.

Có thể thực hiện được những điều sau đây:

- Tiền / Hậu xử lí request.
- Thay đổi giá trị trả về của hàm.
- Thay đổi ngoại lệ được đưa ra từ hàm.
- Mở rộng hành vi của hàm
- Ghi đè hàm

```ts
@Injectable() // @Injectable() デコレータの適用
export class LoggingInterceptor implements NestInterceptor {
  // NestInterceptor  インターフェースの実装
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log("Before...");

    const now = Date.now();
    return next.handle().pipe(
      tap(() => console.log(`After... ${Date.now() - now}ms`)) // レスポンスが返るまでの経過時間を表示
    );
  }
}
```

Nó được sử dụng ở method, controller, global levels

```ts
@Post()
@UseInterceptors(LoggingInterceptor) // Interceptor を登録
async create(@Body() createCatDto: CreateCatDto) {
  // ...
}
```

```ts
@UseInterceptors(LoggingInterceptor)
export class CatsController {}
```

```ts
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(LoggingInterceptor);
```

## Về repository

Repository trong một project NestJS sẽ đảm nhận nhiệm vụ giao tiếp với DB cũng như tiến hành chỉnh sửa, thêm mới dữ liệu.

Ngoài ra việc giao tiếp với các hệ thống bên ngoài cũng do repository đảm nhận

## Tổng kết

Gom tất cả các thành phần đã kể trên lại, chúng ta có thể phác hoạ ra mô hình tổng quan của một ứng dụng sử dụng NestJS như sau:

![Screen Shot 2023-10-09 at 12 42 09](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/cb55cdf8-3e00-46a6-99ff-99dc889c4cdb)

Cảm ơn bạn đọc đã nồng nhiệt đón nhận bài viết của tôi, hi vọng bài viết đã đem lại cho bạn đọc một cái nhìn tổng quan nhất về một ứng dụng NestJS.

Hẹn gặp lại bạn đọc ở những bài viết kế tiếp, xin chân thành cảm ơn.

Happy Learning.
