# Tôi đã xây dựng một API đơn giản bằng DDD như thế nào ? - Phần 2

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
