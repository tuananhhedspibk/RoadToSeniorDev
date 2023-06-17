# Tôi đã xây dựng API bằng DDD như thế nào ? - Phần 2

Tiếp nối [phần 1](link) về DDD trong bài viết phần 2 lần này tôi xin phép được giới thiệu đến bạn đọc các kiến trúc thường dùng với DDD cũng như các khái niệm khác xoay quanh domain trong DDD, rất mong bạn đọc đón nhận nồng nhiệt.

## Các kiến trúc thường dùng với DDD

Khi áp dụng tư tưởng của DDD vào thiết kế hệ thống ta thường sẽ bắt gặp những kiểu kiến trúc phổ biến như sau:

- Kiến trúc 3 tầng (3 layers architecture).
- Kiến trúc phân tầng (Layered architecture).
- Kiến trúc "củ hành" (Onion architecture).
- Kiến trúc Hexagonal (Hexagonal architecture - Port and Adapter architecture).
- Kiến trúc "sạch" (Clean architecture).

Chúng ta sẽ đi lần lượt các kiểu kiến trúc phía trên một cách tổng quan và tóm lược nhất có thể.

### Kiến trúc 3 tầng (3 layers architecture)

Phác thảo qua về hình dáng của nó:

![Screen Shot 2023-06-02 at 7 56 43](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/270a1d04-52ee-45c6-bb8c-1ef8156a0828)

Đây là một kiến trúc phổ biến được nhiều hệ thống web sử dụng, ngoài việc phổ biến thì việc triển khai nó cũng không quá khó thế nhưng nó lại có những nhược điểm lớn như sau:

- Các tầng sẽ có tính ràng buộc và phụ thuộc nhau rất lớn dẫn đến việc khó bảo trì hoặc làm mới từng tầng.
- Tính liên kết với một business logic class là rất thấp.

Để minh hoạ cho điều trên chúng ta cùng tìm hiểu ví dụ sau:

Giả sử ta có một hệ thống quản lí task với các chức năng cơ bản như sau:

- User không thể đăng kí nhiều mail cùng 1 lúc.
- User khi được tạo mới sẽ có trạng thái là `USING` nhưng cũng có thể chuyển đổi thành `SUSPENDING`.
- Task chỉ có thể được gán cho các user có trạng thái là `USING`.
- Task chỉ có 2 trạng thái là `COMPLETE` hoặc `DOING`.

Sơ đồ use-case của nó sẽ như sau:

![Screen Shot 2023-06-04 at 16 43 41](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/4d7726a4-f39a-4937-bae9-9914efe65044)

Rõ ràng ta thấy 2 domain knowledges knowledge `User` và `Task` hoàn toàn không liên quan đến nhau, do đó việc viết chúng vào chung một `business logic layer` sẽ làm giảm **tính liên kết**

Nói sơ qua về **tính liên kết** thì đây là một metric để đánh giá "chất lượng" của 1 class. Ta lấy ví dụ như sau:

```ts
class OperationUntil {
  private counter: number;

  public increment(): void {
    counter++;
  }

  public greet(): void {
    console.log("Hello world");
  }
}
```

Ta thấy method `greet` hoàn toàn không liên quan gì đến thuộc tính `counter` của class, hơn nữa tên của class là `OperationUntil` cũng là một cái tên rất chung chung không rõ ràng về mặt ý nghĩa. Do đó `method` và `property - thuộc tính` của class KHÔNG CÓ SỰ LIÊN HỆ GÌ VỚI NHAU CẢ. Nên class `OperationUntil` này có thể được coi là có tính liên kết thấp.

```ts
class Counter {
  private count: number;

  public increment(): void {
    count++;
  }

  public getCurrentCount(): number {
    return count;
  }
}
```

`getCurrentCount` và `count` giờ đây đã có liên quan tới nhau và cả class `Counter`, nên ta có thể kết luận rằng class này có tính liên kết cao.

Trở lại với ví dụ về `User` và `Task` ở trên, việc ta thực thi domain logic ở tầng **Data Access** là một điều hoàn toàn không hợp lí là vì tầng này sẽ đảm nhận 2 nhiệm vụ:

- Thực thi business logic (model)
- Thực thi xử lí liên quan đến DB (table)

Nên sẽ làm cho 2 tầng `business logic` và `data access` có sự ràng buộc nhất định với nhau.

Tính `ràng buộc` ở đây thể hiện sự phụ thuộc lẫn nhau giữa các layers, classes

Ta lấy ví dụ:

```ts
class Counter {
  public static count: number = 0;
  public static increment(): void {
    count++;
  }
}

class Printer {
  public static print(): void {
    console.log(Counter.count);
  }
}
```

Rõ ràng ta thấy method print của class Printer phụ thuộc vào thuộc tính count của class Counter.

```ts
class Counter {
  public static count: number = 0;
  public static increment(): void {
    count++;
  }
}

class Printer {
  public static print(input: number): void {
    console.log(input);
  }
}

Printer.print(Counter.count);
```

Ta thấy rằng việc truyền tham số cho method print của class Printer đã làm giảm đi sự phụ thuộc lẫn nhau của 2 classes Printer và Counter.

### Kiến trúc phân tầng (Layered architecture)

![Screen Shot 2023-06-04 at 21 30 24](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/d25f6d61-3157-45df-9133-1f156473751b)

Ở kiến trúc này ta sẽ tách tầng `Bussiness Logic Layer` thành 2 tầng:

- `Application Layer` - thực thi usecase
- `Domain Layer` - thực thi domain logic

Ở kiến trúc này:

- Sự liên kết giữa các tầng đã cao hơn.
- Sự phụ thuộc giữa các tầng đã giảm đi nhưng **tầng domain vẫn phụ thuộc vào tầng infra** (phụ thuộc vào việc sử dụng DB hay OR Mapper) và đây là điều cần tránh

### Kiến trúc "củ hành" (Onion architecture)

Cũng gần giống với kiến trúc phân tầng ở trên nhưng **sự phụ thuộc của tầng domain vào tầng infra đã bị triệt tiêu hoàn toàn**

![Screen Shot 2023-06-04 at 21 36 34](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/23594212-2b01-46f9-9faa-e4d314f5bdea)

Cụ thể là tầng domain sẽ định nghĩa các `Repository Interface`, tầng infra sẽ "implement" các interfaces nêu trên. Nên lúc này tầng infra sẽ phụ thuộc vào tầng domain. Tầng infra cũng sẽ tiến hành việc lưu `Domain Aggregate` vào trong DB.

Đặc điểm của các tầng còn lại (Presentation, Usecase, Domain) sẽ như sau:

- Tầng domain: cần độc lập, không phụ thuộc vào bất kì tầng nào. Chứa các `Value-Object`, `Domain Aggregate`, `Domain Event`
- Tầng usecase: sử dụng các public method của các `Domain Aggregate`, tầng này không được phép phụ thuộc vào tầng presentation
- Tầng presentation: tương tác trực tiếp với client, nó chứa các classes: `Controller`, `External-Controller`

![Onion Architecture](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/abeae8ed-5be8-4acb-8efc-ef4341d76260)

Việc các Actors bên ngoài tới Application cũng như nhận req từ Application đều thông qua các Adapters.

### Kiến trúc Hexagonal (Hexagonal architecture - Port and Adapter architecture)

Tư tưởng chính ở đây đó là Application sẽ tương tác với thế giới bên ngoài thông qua:

- Adapter
- Port chuyên dụng

Tóm tắt về kiến trúc này sẽ như sau:

![Hexagonal Architecture](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/b260a6c3-bb1f-4692-839d-7ba252b79e70)

### Kiến trúc "sạch" (Clean architecture)

![Clean Architecture](https://user-images.githubusercontent.com/15076665/175429364-68f2d02f-2956-4278-8ea6-c84ae3377139.png)

Kiến trúc này là sự tổng hợp và kế thừa từ `Onion Architecture` và `Hexagonal Architecture`.

Cách đặt tên từng tầng có thể có đôi chút khác biệt:

- Usecase Layer → Application layer
- Adapter Layer → Interface Adapter Layer

## Các khái niệm xoay quanh tầng domain

Ngoài khái niệm về domain aggregate như đã nói ở phần trước, chúng ta còn có các khái niệm khác ở tầng domain như:

- Value-Object
- Domain Service
- Repository
- Factory

### Value Object

Value-Object là một loại giá trị mà nó sẽ không hề có ý nghĩa khi không gắn với bất kì một Aggregate cụ thể nào cả. Ta lấy ví dụ:

Trong công ty, mỗi một nhân viên sẽ có một **mã số nhân viên** của riêng mình (VD: 1234), và mã số này sẽ chỉ gắn với một nhân viên duy nhất mà thôi. Mã số này sẽ được sử dụng để:

- Quản lí tiến độ công việc
- Quản lí nhân sự
- ...

nghĩa là con số **1234** này có ý nghĩa rất quan trọng đối với công ty và chính nhân viên đó. Thế nhưng nếu nó không gắn với bất kì một nhân viên nào thì nó chỉ đơn thuần là **một con số** mà thôi, đồng thời nó cũng không biểu đạt một business logic nào cả.

Trong API của mình tôi có định nghĩa một `class EmailValueObject` như sau: <https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/blob/main/src/domain/value-object/email-vo.ts>

Với email thông thường, chỉ cần tuân thủ đúng format **xxx@domain** là đủ, nhưng có thể do đặc thù của hệ thống, bạn có thể thêm những điều kiện khác cho email (VD: phải bắt đầu bằng chữ cái thường, ...), những điều kiện trên chính là các business logic thuộc về tầng domain.

Ở API của mình, tôi quy định email phải tuân thủ theo một format được kiểm tra bởi regex đã định nghĩa sẵn như sau:

```ts
const mailRegex =
  /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
const mailSchema = z.string().regex(mailRegex);

try {
  mailSchema.parse(input);
  this.value = input;
} catch (e) {
  throw new DomainError({
    code: DomainErrorCode.BAD_REQUEST,
    message: 'Invalid email format',
    info: {
      detailCode: DomainErrorDetailCode.INVALID_EMAIL_FORMAT,
    },
  });
}
```

### Domain Service

Dùng khi **việc biểu thị model bằng một object là không thể**. Thông thường sẽ thao tác với **một tập các objects**.

VD: một ví dụ tiêu biểu đó là việc check xem mail có bị trùng lặp hay không - nói cách khác mail đã được sử dụng cho user trong hệ thống hay chưa. Bản thân một user object có thể biết được mail của mình nhưng không thể biết được thông tin về mail của object khác nên việc tự nó kiểm tra là điều không thể.

Những trường hợp như trên thường sẽ được xử lí bởi `domain service`.

Thế nhưng:

> Hãy cố gắng sử dụng entity và value-object nhiều nhất có thể và hạn chế tối đa việc sử dụng domain-service

Lí do là bởi nếu vô tình viết nhiều business logic vào đây thì trong tương lai nó sẽ trở thành một Fat class một cách không mong muốn.

### Repository

Dùng để lưu dữ liệu của `Aggregate` vào DB. Thông thường 1 aggregate sẽ tương ứng với một Repository.

Việc truyền aggregate vào repository hay trả về aggregate đều phải thông qua `root aggregate`. Ở API của mình, tôi tạo ra `UserRepository` như sau:

Đầu tiên, tôi định nghĩa một `abstract class IUserRepository` ở tầng domain.

```ts
export abstract class IUserRepository {
  getByEmail: (
    transaction: TransactionType | null,
    email: string,
  ) => Promise<UserEntity | null>;
  save: (transaction: TransactionType, user: UserEntity) => Promise<UserEntity>;
}
```

Nó định nghĩa 2 method signature là `getByEmail` & `save` với chức năng lần lượt là:

- `getByEmail`: lấy user từ DB thông qua email.
- `save`: lưu dữ liệu của user vào DB.

Tôi triển khai (implement) abstract class này ở tầng infra như sau:

```ts
class UserRepository implements IUserRepository {
  async getByEmail(
    transaction: Transaction | null,
    email: string,
  ): Promise<DomainUserEntity | null> {
    const repository = transaction
      ? transaction.getRepository(RDBUserEntity)
      : getRepository(RDBUserEntity);

    const query = this.getBaseQuery(repository);
    const user = await query.where('user.email = :email', { email }).getOne();

    return userFactory.createUserEntity(user);
  }

  async save(
    transaction: TransactionType,
    user: DomainUserEntity,
  ): Promise<DomainUserEntity> {
    const repository = transaction
      ? transaction.getRepository(RDBUserEntity)
      : getRepository(RDBUserEntity);

    const salt = randomlyGenerateSalt();
    const passwordHashedWithSalt = hashPassword(user.password.toString(), salt);

    const createdUser = await repository.save({
      email: user.email.toString(),
      password: passwordHashedWithSalt,
      userName: user.userName,
      salt,
    });

    return userFactory.createUserEntity(createdUser);
  }
}
```

Các bạn có thể tham khảo full source code của `abstract class` tại [đây](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/blob/main/src/domain/repository/user.ts) và `implement class` tại [đây](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/blob/main/src/infrastructure/repository/user/index.ts).

Lí do tôi sử dụng abstract class chứ không phải interface ở đây đó là: tôi muốn `IUserRepository` ở tầng domain sẽ giống như một `BaseClass` của domain user để từ đó tôi có thể tạo thêm các `SubBaseClass` của domain user khác như `IAdminRepository` hoặc `IMemberRepository`.

### Factory

Thường được sử dụng để tạo ra object mới khi logic tạo ra object khá phức tạp. Bản thân factory cũng được coi như một loại `domain service`.

Tôi lấy ví dụ với API do mình phát triển như sau:

```ts
import { plainToClass, ClassConstructor } from '@nestjs/class-transformer';

export class UserFactory {
  protected createEntity<E, P>(entity: ClassConstructor<E>, plain: P): E {
    return plainToClass(entity, plain, {
      excludeExtraneousValues: true,
    }) as E;
  }

  createUserEntity(user: User | null) {
    // user param has User type (same structure with DB table)
    if (!user) return null;

    const entity = this.createEntity(UserEntity, {
      ...user,
      detail: user.userDetail || null,
    });

    return entity;
  }
}
```

Ở đây tôi sử dụng method `createUserEntity` để tạo ra các UserEntity dùng cho tầng domain, với tham số đầu vào là `user` có kiểu dữ liệu là `User` - kiểu dữ liệu này có cấu trúc giống hệt `User Model` hay nói cách khác nó chính là dữ liệu mà tôi lấy trực tiếp từ DB ra.

Trong method `createUserEntity` tôi sử dụng hàm `plainToClass` của thư viện `class-transformer` với mục đích "ép" cho `User Model` trở thành `UserEntity` để có đầu ra như ý muốn của mình.

## Kết phần hai

Vậy là trong phần hai này tôi đã trình bày được với bạn đọc các kiểu kiến trúc thường dùng với DDD đó là:

- Kiến trúc 3 tầng (3 layers architecture).
- Kiến trúc phân tầng (Layered architecture).
- Kiến trúc "củ hành" (Onion architecture).
- Kiến trúc Hexagonal (Hexagonal architecture - Port and Adapter architecture).
- Kiến trúc "sạch" (Clean architecture).

cũng như các khái niệm quan trong khác thường được đề cập đến trong tầng domain của DDD như:

- Value-Object
- Domain Service
- Repository
- Factory

Do hạn chế về mặt kiến thức cũng như phạm vi của blog nên tôi chỉ có thể trình bày với bạn đọc những nội dung mang tính chất tóm lược và quan trọng nhất, hi vọng chúng sẽ ít nhiều hữu ích cho bạn đọc sau này. Xin trân thành cảm ơn bạn đọc và hẹn gặp lại ở phần ba trong series các blog về DDD.
