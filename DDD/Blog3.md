# Tôi đã xây dựng một API đơn giản bằng DDD như thế nào ? - Phần 3

Chào mừng bạn đọc đã quay trở lại phần 3 trong series về DDD của tôi, bạn đọc có thể xem lại:

- Phần 1 [tại đây](link)
- Phần 2 [tại đây](link)

Trong phần 3 này, tôi xin phép được trình bày với bạn đọc về 2 nội dung chính sau:

1. Phân chia context.
2. Kiến trúc của tầng usecase.

hi vọng sẽ được bạn đọc đón nhận một cách nồng nhiệt nhất. Xin cảm ơn.

## Phân chia context

Nói về việc phân chia context, một ý tưởng rất đơn giản mà ai cũng có thể nghĩ đến đó là **1 context - 1 application**, điều này đồng nghĩa với việc **triển khai microservice**.

Cách làm này có ưu điểm về tính mở rộng khi hệ thống trở nên "phình to", thế nhưng nó lại tốn nhiều chi phí cũng như khó để triển khai.

### 1 context - 1 application

Ta xét một ứng dụng EC, về cơ bản ta có thể phân chia ứng dụng này thành 2 contexts:

- Seller: phục vụ cho việc bán hàng.
- Logistic: phục vụ việc quản lí kho vận cũng như hàng hoá.

Với 2 contexts như vậy ta có thể thấy rằng, với cùng một sản phẩm (product) nhưng mỗi context sẽ quan tâm đến các khía cạnh khác nhau.

**Với Seller** sẽ là `giá tiền - price`, `số lượng hàng trong kho - stock_count`.

**Với Logistic** sẽ là `trạng thái vận chuyển - shipping_status`, `địa chỉ đến - shipping_address`.

Kiến trúc của hệ thống với 2 contexts có thể mô tả một cách đơn giản như sau:

![Screen Shot 2023-06-14 at 22 57 13](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/assets/15076665/421a2dd7-0945-4326-8035-3da55f70b020)

Về cơ bản sự tương tác giữa 2 contexts ở đây đó là:

- Đồng bộ: thông qua lời gọi trực tiếp như REST API call.
- Bất đồng bộ: thông qua việc gửi các events như AWS SQS.

### n contexts - 1 application

Ta sẽ tiến hành phân chia các folders tương ứng với các contexts như ví dụ dưới đây

```text
app
--- delivery
------- domain
------- infra
------- repo

--- sale
------- domain
------- infra
------- repo
```

> Một điều cần phải chú ý ở đây đó là cần phải phân chia folder cẩn thận ngay từ đầu để dễ dàng chia nhỏ app sau này khi nó phình to

## Kiến trúc tầng usecase

### Nhiệm vụ của tầng use-case

Như tên gọi của mình, bản thân tầng use-case sẽ có nhiệm vụ thực thi các nghiệp vụ (chức năng) chính của hệ thống. Tôi lấy ví dụ với API của mình, cụ thể là các chức năng chính phía users bao gồm:

- Follow
- Unfollow
- Update Password
- Update Profile

nên tầng usecase của tôi sẽ bao gồm các use-cases chính như hình bên dưới

![Screen Shot 2023-06-15 at 8 06 41](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/assets/15076665/5d23581f-5b01-442a-9861-37e5d1885b43)

Các bạn có thể tham khảo thêm [tại đây](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/tree/main/src/usecase/user)

Thử phân tích một use-cases cụ thể đó là `UpdateProfile` - [source code](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/blob/main/src/usecase/user/update-profile/index.ts)

```ts
const userEntity = await this.userRepository.getById(null, userId);

if (input.email) {
  userEntity.updateEmail(input.email);
}

if (input.userName) {
  userEntity.updateUserName(input.userName);
}
```

Phía trên là một phần code nhỏ trong use-case `UpdateProfile`, ta có thể thấy rằng use-case là nơi ta sử dụng các `domain entities`, có thể là để chúng tương tác với nhau, nhưng cũng có thể là sử dụng các **public method** được định nghĩa sẵn (những public method này sẽ phản ánh đúng nghiệp vụ mà User Domain đảm nhiệm) để từ đó thực thi một tính năng trong hệ thống - ở đây là tính tăng `UpdateProfile` hay cập nhận thông tin của user.

> Việc sử dụng các method của domain entity sẽ làm tăng tính trừu tượng cho tầng domain khi tầng use-case chỉ quan tâm đến việc sử dụng method mà không cần phải quan tâm cách triển khai hay nội dung bên trong của nó là gì

### Giá trị trả về từ tầng use-case

Như đã phân tích trong [phần 2](link), nếu sử dụng **Onion Architecture**, hệ thống của chúng ta có thể được sơ đồ hoá như sau:

![Onion Architecture](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/abeae8ed-5be8-4acb-8efc-ef4341d76260)

Ta thấy rằng **tầng presentation** sẽ bao ngoài **tầng usecase**, do đó nó sẽ nhận giá trị trả về từ tầng usecase.

Về việc truyền giá trị trả về từ tầng use-case xuống tầng presentation, ta có 2 cách làm như sau:

1. Tạo một class chuyên dùng để chứa kiểu dữ liệu truyền xuống này
2. Truyền nguyên domain object xuống tầng presentation

Ta cùng phân tích ưu nhược điểm của 2 cách làm trên.

Với **cách 1**, ưu điểm của nó có thể liệt kê ra như sau:

- Tầng presentation sẽ không thể nhìn thấy được các domain business method của tầng domain.
- Tránh việc sửa đổi tầng domain làm ảnh hưởng tới tầng presentation.

Ở API của mình tôi sử dụng cách tiếp cận này. Trở lại ví dụ về `UpdateProfile` usecase ở trên, tôi định nghĩa một class output riêng cho usecase này như sau:

```ts
const ApiResultCode = {
  OK: 'OK',
  WARN: 'WARN',
  ERROR: 'ERROR',
} as const;

export default class ApiResultDto extends BaseDto {
  code: ApiResultCode;
  warnList: ApiWarn[];
  errorList: ApiError[];
  message?: string;
}


class UpdateUserProfileUsecaseOutput extends UsecaseOutput {
  @ApiProperty({
    description: 'API result',
    type: ApiResultDto,
    required: true,
  })
  result: ApiResultDto;
}
```

Bạn đọc có thể tham khảo thêm tại [đây](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/blob/main/src/usecase/user/update-profile/index.ts#L68)

Output trong usecase này sẽ chỉ thuần tuý là trả về kết quả thực thi usecase thành công hay thất bại thông qua thuộc tính `code: ApiResultCode`, rất đơn giản như sau:

```ts
const output = new UpdateUserProfileUsecaseOutput();
output.result = ApiResultDto.ok();

return output;
```

Ưu điểm là như vậy nhưng nhược điểm của cách làm này đó chính là chi phí để chuyển đổi kiểu dữ liệu sẽ tăng.

Với **cách 2** thì ưu cũng như nhược điểm của nó ngược lại hoàn toàn so với **cách 1**.

Nếu thực thi cách thứ 2 thì domain object cần có các **method phục vụ cho việc hiển thị**, từ đó sẽ làm cho domain object bị phình to, khó bảo trì. Vậy nên nếu áp dụng cách 2 thì thay vì tốn công chuyển đổi dữ liệu thì cái giá phải trả cũng lớn không kém.

### Đặt tên cho Return Value Class

Bạn có thể đặt tên theo format **XXXDTO** - DTO là **Data Transfer Object**

![File_000 (2)](https://user-images.githubusercontent.com/15076665/176991118-62750bda-4af4-42a0-850b-54f83435f77a.png)

## Một vài chú ý khác khi triển khai tầng use-case

### Phân chia class ở tầng use-case

Thông thường sẽ là **1 class - 1 public method**, trong API của mình tôi thường cấu trúc cho một use-case class của mình như sau:

```ts
export default class UpdateUserProfileUsecase {
  async execute(input) {
    // ...
  }
}
```

bạn đọc có thể tham khảo thêm [tại đây](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/blob/main/src/usecase/user/update-profile/index.ts)

### Cách đặt tên cho use-case class

- Với các classes có 1 method duy nhất thì ta **giữ nguyên động từ** - VD: **CreateTaskUseCase**.
- Với các classes có nhiều methods thì ta **bỏ động từ khỏi tên class** - VD: **TaskUseCase**.

Về cơ bản việc đặt tên cho use-case class sẽ tuỳ vào nội dung cũng như nghiệp vụ của từng project riêng, thế nhưng một trong những điều ta cần phải chú ý ở đây đó là việc nên nghĩ tên cho use-case class ngay từ khâu thiết kế sơ đồ use-case.

## Kết phần 3

Vậy là phần 3 trong series về DDD của tôi đã khép lại, trong phần 3 này tôi đã chia sẻ được với bạn đọc về:

1. Phân chia context.
2. Kiến trúc của tầng usecase.

Hẹn gặp lại các bạn vào phần 4 của series về DDD, trong phần 4 tôi sẽ nói về việc triển khai tầng presentation và tầng infrastructure, rất mong bạn đọc đón nhận một cách nồng nhiệt.
