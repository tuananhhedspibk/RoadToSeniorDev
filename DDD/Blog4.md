# Tôi đã xây dựng API bằng DDD như thế nào ? - Phần 4

Chào mừng bạn đọc đã quay trở lại phần 4 trong series về DDD của tôi, bạn đọc có thể xem lại:

- Phần 1 [tại đây](link)
- Phần 2 [tại đây](link)
- Phần 3 [tại đây](link)

Trong phần 4 này, tôi xin phép được trình bày với bạn đọc về 2 nội dung chính sau:

1. Cách triển khai tầng presentation.
2. Cách triển khai tầng infrastructure.

hi vọng sẽ được bạn đọc đón nhận một cách nồng nhiệt nhất. Xin cảm ơn.

## Cách triển khai tầng presentation

### Xử lí của tầng presentation

Đây sẽ là tầng tương tác trực tiếp với client, do đó ở giữa tầng presentation và tầng usecase cần thực hiện convert data.

![File_000](https://user-images.githubusercontent.com/15076665/178100102-b6d54ae4-7634-4615-9889-5b1e9f528afd.png)

Trong ứng dụng sẽ có:

- JSON controller nhận và trả về kết quả dưới format json
- HTML controller nhận dữ liệu submit từ HTML form thông qua format **application/x-www-form-urlencoded**

Thông thường tầng presentation sẽ chứa các classes dạng `XXXController`, do nó đảm nhận nhiệm vụ tương tác trực tiếp với client nên ngoài controller thì các routing file cũng có thể được đưa vào đây.

Với API của mình tôi dành ra một folder riêng cho tầng presentation, trong đó sẽ chứa các controllers như:

- authentication
- user
- post
- ...

![Screen Shot 2023-06-17 at 11 15 01](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/cd2ae42e-2632-4a47-8b79-17f8bc52dca0)

Các bạn có thể tham khảo thêm [tại đây](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/tree/main/src/presentation/internal)

Và tuỳ theo từng framework, tầng này có thể chứa các middlewares như `logging` hay `authentication`, ... Tôi sẽ không đi sâu vào các middlewares này do chúng không thuộc phạm vi của bài viết lần này.

Trong một controller, tương ứng với mỗi route sẽ là **một use-case** đảm nhận việc xử lí các nghiệp vụ liên quan đến route đó. Lấy ví dụ với `API update post`

```ts
@Controller('post')
class PostController {
  @Put('/update')　// route: /post/update
  update(
    @Body() payload: UpdatePostUsecaseInput,
    @Req() request: { user: { userId: number } },
  ) {
    return this.updatePostUsecase.execute(payload, request.user.userId);
  }
}
```

Ở đây tôi có tính năng API `post/update` sẽ được đảm nhận bởi `updatePostUsecase`. Use-case này sẽ nhận đầu vào gồm:

- userId được lấy từ request gửi lên từ phía client.
- payload với kiểu dữ liệu là `UpdatePostUsecaseInput` cho chính usecase này quy định trước.

### Định nghĩa cấu trúc của response

Tầng presentation cũng sẽ quan tâm đến format của response trả về cho phía client.

![Screen Shot 2023-06-17 at 11 29 11](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/47c17a01-7d54-47bf-b1af-017819f9de0c)

Như hình ví dụ trên, client muốn hiển thị `1,000` thay vì `1000` thì nhiệm vụ convert format cho dữ liệu trả về này sẽ là nhiệm vụ của `tầng presentation`

## Cách triển khai tầng infrastructure

Đây là tầng nằm ngoài cùng trong kiến trúc "củ hành" đã trình bày ở [phần 2](link). Tôi cũng sẽ minh hoạ lại nó trong bài viết lần này

![Screen Shot 2023-06-17 at 11 50 19](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/701799c4-941c-4799-911e-c622846ec828)

Như ở hình vẽ trên, tầng infrastructure sẽ làm những nhiệm vụ:

- Tương tác với các hệ thống hoặc API ngoài.
- Tương tác trực tiếp với DB.

Vậy câu hỏi đặt ra ở đây là tại sao chúng ta lại phải triển khai các xử lí tương tác với hệ thống ngoài ở tầng infastructure. Rất đơn giản đó là trong DDD ta luôn muốn tầng domain (trung tâm của hệ thống) sẽ **ít phụ thuộc** vào công nghệ sử dụng nhất có thể.

Vì mỗi lần thay đổi công nghệ sử dụng (VD: thay đổi từ PostgreSQL sang MySQL) thì tầm ảnh hưởng của nó lên hệ thống là rất lớn, do đó tầng domain hay thậm chí cả tầng usecase là những tầng:

- Triển khai core logic hoặc business logic của hệ thống (tầng domain).
- Triển khai các nghiệp vụ, tính năng của hệ thống (tầng usecase).

sẽ cần phải được bảo vệ khỏi những "sự ảnh hưởng mang tính chất công nghệ" như trên, đồng thời mỗi lần thay đổi nghiệp vụ hoặc business logic thì chắc chắn khả năng cao là toàn bộ hệ thống cũng sẽ bị ảnh hưởng theo.

Nên tư tưởng ở đây đó là:

- Định nghĩa các repository interfaces ở tầng domain.
- Triển khai các repository interfaces này ở tầng infrastructure.
- Tầng usecase sẽ sử dụng các `method signatures thuộc về interface` (tầng domain) thay vì sử dụng `instance method của implementing class` (tầng infrastructure). Vì đơn giản usecase chỉ cần quan tâm đến đầu vào và đầu ra của method chứ KHÔNG CẦN PHẢI QUAN TÂM đến cách thức triển khai cũng như nội dung bên trong của method, việc làm này sẽ giúp **tăng tính độc lập của tầng use-case** và **giảm đi tính liên kết** giữa các tầng.

Tôi lấy ví dụ ở API của mình. Với `user domain` tôi có định nghĩa `UserRepository interface` ở tầng domain như sau:

```ts
export abstract class IUserRepository extends BaseRepository {
  getById: (
    transaction: TransactionType | null,
    id: number,
  ) => Promise<UserEntity | null>;
  update: (
    transaction: TransactionType,
    user: UserEntity,
  ) => Promise<UserEntity>;
}
```

Full source code: <https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/blob/main/src/domain/repository/user.ts>

Và tầng usecase sẽ sử dụng các methods đó như sau:

```ts
export default class UpdateUserProfileUsecase {
  constructor(@Inject(IUserRepository) private readonly userRepository: IUserRepository) {}

  async execute(input, userId) {
    const userEntity = await this.userRepository.getById(null, userId);

    userEntity.updateEmail(input.email);

    await this.transactionManager.transaction(
      async (transaction: TransactionType): Promise<void> => {
        await this.userRepository.update(transaction, userEntity);
      },
    );
  }
}
```

ta thấy rằng tầng usecase chỉ cần "inject" `UserRepository Interface` và sau đó sử dụng các `method signature` của nó như `getById` hoặc `update`. Chi tiết hơn về "inject" `UserRepository Interface` mà cụ thể là `Dependencies Injection` các bạn có thể tìm hiểu thêm [tại đây](https://viblo.asia/p/dependency-injection-trong-typescript-aWj53m1eZ6m).

Các bạn thấy rằng các method thuộc về `UserRepository interface` sẽ đều trả về các `UserDomainEntity` nên do đó các implementing class nằm trong tầng infrastructure ngoài việc lấy "raw data" từ DB ra, cũng cần phải "convert" nó sang dạng `DomainEntity` và ở đây chúng ta sẽ sử dụng factory (như đã trình bày ở [phần 1](link)) để tiến hành việc convert.

Việc triển khai ở tầng infrastructure sẽ như sau:

```ts
class UserRepository implements IUserRepository {
  async getById(
    transaction: TransactionType,
    id: number,
  ): Promise<DomainUserEntity> {
    const repository = transaction
      ? transaction.getRepository(RDBUserEntity)
      : getRepository(RDBUserEntity);

    const query = this.getBaseQuery(repository).where('user.id = :id', { id });
    const user = await query.getOne();

    return userFactory.createUserEntity(user);
  }
}
```

Phân tích sơ qua thì đầu tiên ta sẽ lấy dữ liệu từ DB ra bằng cách sử dụng Repository của typeorm

```ts
const query = this.getBaseQuery(repository).where('user.id = :id', { id });
const user = await query.getOne();
```

`user` lúc này sẽ chứa "raw data" thuần tuý và để thu về được `UserDomainEnttiy` ta sẽ cần sử dụng factory như sau:

```ts
userFactory.createUserEntity(user);
```

## Kết phần 4

Vậy là trong bài viết lần này tôi đã trình bày được với bạn đọc những ý chính về:

1. Cách triển khai tầng presentation.
2. Cách triển khai tầng infrastructure.

và đây cũng là bài viết cuối cùng trong series về DDD của tôi, do hạn chế về mặt kiến thức nên các bài viết không thể tránh khỏi những thiếu sót, tôi hi vọng sẽ nhận được những ý kiến đóng góp cũng như phản biện của bạn đọc để có thể liên tục cập nhật, làm mới các bài viết trong series về DDD lần này của mình.

Xin chân thành cảm ơn bạn đọc.
