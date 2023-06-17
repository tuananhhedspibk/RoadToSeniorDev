# Tôi đã xây dựng API bằng DDD như thế nào ? - Phần 1

DDD đã và vẫn đang là một xu hướng thiết kế được ưa chuộng trong các hệ thống lớn, mang nặng tính nghiệp vụ ở thời điểm hiện tại. Thế nhưng việc hiểu rõ những định nghĩa mang tính "hàn lâm" của DDD lại rất mất thời gian, và để hỗ trợ cho các lập trình viên nói chung cũng như những ai đang có ý định tìm hiểu về DDD nói riêng có thể rút ngắn thời gian "thẩm thấu" những kiến thức "hàn lâm" của DDD tôi xin phép được trình bày với bạn đọc những kiến thức rất cơ bản cũng như những ví dụ minh hoạ thông qua series bài viết về DDD lần này, rất mong được bạn đọc đón nhận.

## Tổng quan về API mà tôi đã xây dựng

Tôi xây dựng API này với mục đích chính là để thực hành thêm về DDD nên sẽ không quá chú trọng vào các chức năng phức tạp. Về cơ bản API của tôi có thể được mô tả bằng sơ đồ usecase như sau:

![Screen Shot 2023-06-01 at 8 06 37](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/941b8d88-cdcd-46b5-918f-3fa7b8d022ce)

Về cơ bản đây là một API cho phép người sử dụng có thể:

- Tạo bài đăng với ảnh đi kèm
- Comment / Like bài đăng
- Follow / Unfollow những người dùng khác

Bạn đọc có thể tham khảo source code của tôi ở địa chỉ: <https://github.com/tuananhhedspibk/NewAnigram-BE-DDD>

Trong API này, tôi sử dụng hai công nghệ chính đó là `nestjs` framework và `mysql` database.

## DDD là gì ?

DDD (Domain Driven Design) là một cách thiết kế phần mềm "hướng nghiệp vụ" - tức sẽ lấy nghiệp vụ (domain) làm trung tâm của hệ thống.

Có thể ví dụ với hệ thống "nghiệp vụ ngân hàng" hoặc "nghiệp vụ kho vận". Nếu chỉ nói về mặt khái niệm thì sẽ rất khó hình dung DDD là gì nên do đó trong bài viết này tôi sẽ cố gắng minh hoạ từng khái niệm của DDD tương ứng với code API của mình.

### Một vài khái niệm cơ bản trong DDD

Vì là "domain" driven design nên **domain** là khái niệm chính ở đây. Xoay quanh khái niệm về domain ta có 3 key words chính đó là **Domain**, **Model** và **Domain Model**

**Domain** là một lĩnh vực nghiệp vụ nào đó ví dụ như "nghiệp vụ ngân hàng" hoặc "nghiệp vụ kho vận", ...

**Model** là quá trình trừu tượng hoá các đối tượng cũng như các thao tác của nghiệp vụ. Quá trình mô hình hoá nghiệp vụ thành các đối tượng mang tính trừu tượng này được gọi là quá trình **Modeling** hoặc **Mô hình hoá**.

**Domain Model** là các models ta thu được sau quá trình **Modeling** ở trên.

Trở lại với API ở trên, domain của tôi chính là việc `người dùng có khả năng đăng bài và theo dõi các người dùng khác`. Tôi sẽ mô hình hoá thành 2 models chính là:

- User
- Post

đây cũng chính là hai "đối tượng" chính trong API của tôi. Minh hoạ rõ ràng hơn cho 2 models trên tôi có sơ đồ Domain Model như sau:

![Screen Shot 2023-06-01 at 8 02 43](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/e438f205-c319-4211-b607-edb7f163105c)

Nhìn qua sơ đồ này bạn đọc có thể thấy được:

- Các thuộc tính cơ bản của post: content, imageUrls, ...
- Các thuộc tính cơ bản của user: email, userName, password
- Mối quan hệ 1-n giữa User-Post

Nó đúng nhưng vẫn còn thiếu rất nhiều, ví dụ như:

- Comment
- Like
- Follow
- User detail (lưu thông tin chi tiết của user như: firstName, lastName, age, ...)

Không những thế trong DDD chúng ta còn có một khái niệm rất quan trọng khác đó là **Aggregate - Kết tập**. Giải thích một cách đơn giản thì aggregate (kết tập) là: một tập hợp các objects có liên quan chặt chẽ với nhau về mặt dữ liệu mà ta luôn phải đảm bảo điều đó.

Lấy ví dụ như sau:

![Screen Shot 2023-06-01 at 8 16 16](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/43e4e501-01aa-46c9-94d7-33bd9d0b19c8)

Một Club sẽ có các "Member", ở đây ta quy định status của Club sẽ là `FULL` nếu giá trị của `memberCount = 5`, status của Club sẽ là `STILL_FREE` nếu giá trị của `memberCount < 5`. Từ quy định này ta thấy rằng giữa Club và Member có một mối liên hệ về mặt dữ liệu khá chặt chẽ nên có thể đưa ra kết luận rằng Club và Member sẽ thuộc cùng một aggregate - kết tập (chú thích: từ nay trở đi tôi sẽ sử dụng thuật ngữ aggregate). Nên sơ đồ domain model của Club và Member sẽ được chỉnh sửa lại như sau:

![Screen Shot 2023-06-01 at 8 20 34](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/45fd9793-169b-45fa-9a6b-58c4d47b8c9f)

Trở lại với API mà tôi đã giới thiệu, sau khi cân nhắc thêm Follow, Like, Comment và User Detail tôi quyết định chia aggregate như sau:

![Screen Shot 2023-06-01 at 21 29 19](https://github.com/tuananhhedspibk/DataIntensiveApp/assets/15076665/3a0741f2-c6af-4b2d-add6-52e2bdfcda09)

Giải thích sơ qua như sau: tôi tiến hành chia domain của mình thành 3 aggregates lần lượt là:

- User
- Post
- Follow

API của tôi không có quá nhiều ràng buộc về mặt dữ liệu như ví dụ về Club và Member ở trên nhưng việc tổ chức theo "đơn vị" aggregate sẽ giúp code thể hiện rõ ràng hơn về nghiệp vụ.

Tôi lấy ví dụ về User Aggregate: trong aggregate này tôi lưu thông tin cơ bản của user (email, password) và thông tin chi tiết (nickName, avatarUrl) của user thông qua 2 objects lần lượt là `User` và `UserDetail`, ở đây `User` sẽ được gọi là `Root Aggregate` hay kết tập gốc.

Một trong những nguyên tắc cơ bản khi làm việc với aggregate đó là chỉ được cập nhật thông tin của aggregate thông qua `root aggregate`, nghĩa là ở đây việc cập nhật thông tin cho `UserDetail` hay `User` đều phải thông qua `User Aggregate` hay nói cách khác là `User Object`.

Bạn có thể xem cách tôi định nghĩa `User Aggreate` tại [địa chỉ](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/blob/main/src/domain/entity/user/index.ts). Bạn có thể thấy cách tôi cập nhật `UserDetail` như sau:

```ts
export class UserEntity extends BaseEntity {
  updateDetail(params: UpdateDetailParams) {
    if (!this.detail) {
      this.detail = new UserDetailEntity();
    }

    if (params.avatarURL) {
      this.detail.avatarURL = params.avatarURL;
    }

    if (params.gender) {
      this.detail.gender = params.gender;
    }

    if (params.nickName !== null) {
      this.detail.nickName = params.nickName;
    }
  }
}
```

tôi tạo một method `updateDetail` bên trong class `UserEntity` (đây chính là User Aggregate class), method này sẽ cập nhật thông tin của user detail nên do đó việc cập nhật user detail trên thực tế trông sẽ như sau:

```ts
const user = new UserEntity(); // Định nghĩa User Aggregate

user.updateDetail({ nickName: 'testUser' }); // Cập nhật thông tin detail thông qua root aggregate là user
```

Và User aggregate sẽ "kết tập" userDetail bên trong nó thông qua thuộc tính của class như sau:

```ts
class UserEntity extends BaseEntity {
  detail: UserDetailEntity;
}

class UserDetailEntity extends BaseEntity {
  id?: number;
  active: boolean;
  nickName: string;
  avatarURL: string;
  gender: UserDetailGender;

  constructor() {
    super();
  }
}
```

Bản thân từng method được định nghĩa trong các Aggregate class sẽ thể hiện cho nội dung của nghiệp vụ mà hệ thống đang tiến hành mô hình hoá, ở ví dụ trên đó là `updateDetail`.

Phân tích tương tự với `Post Aggregate` tại [source code](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/blob/main/src/domain/entity/post/index.ts). `Post Aggregate` với `Post object` là `root aggregate`, các objects `Comment` & `Like` sẽ là các objects con được "kết tập" bên trong `Post Aggregate` thông qua các thuộc tính của class như sau:

```ts
class PostEntity extends BaseEntity {
  likes: LikeEntity[];
  comments: CommentEntity[];
}

class CommentEntity extends BaseEntity {
  id?: number;
  content: string;
  userId: number;
  postId: number;
}

class LikeEntity extends BaseEntity {
  id?: number;
  userId: number;
  postId: number;
}
```

Còn với `Follow Aggregate` thì khá đơn giản, nó chỉ có 2 thuộc tính duy nhất đó là:

- srcUserId: id của user đang tiến hành follow.
- destUserId: id của user đang được follow.

Cụ thể hơn bạn đọc có thể xem tại [đây](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/blob/main/src/domain/entity/follow.ts)

## Kết phần 1

OK, vậy là đã xong một khái niệm hết sức cơ bản và quan trọng của DDD đó là `Aggregate`, hi vọng thông qua bài viết mở đầu này bạn đọc đã thấm được một vài "ngón đòn" đầu tiên của DDD. Về cơ bản sau phần này tôi mong rằng bạn đọc có thể hiểu được:

- Domain là gì ?
- Modeling là gì ?
- Domain Model là gì ?
- Aggregate là gì ? Aggregate có những đặc thù nào đáng phải lưu tâm ?

Ngoài ra còn một nhắc nhở nho nhỏ với bạn đọc đó là hai sơ đồ tôi đã trình bày ở trên đó là `Sơ đồ usecase` và `Sơ đồ domain model`, tôi sẽ không trình bày chi tiết về định nghĩa của 2 loại sơ đồ này nhưng nói một cách đơn giản:

- `Sơ đồ usecase` sẽ thể hiện những chức năng chính mà hệ thống sẽ cung cấp cho người dùng (sơ đồ này phải có **trước** sơ đồ domain model)
- `Sơ đồ domain model` sẽ thể hiện việc chúng ta mô hình hoá các đối tượng trong hệ thống nghiệp vụ của mình ra sao. Trong sơ đồ này bạn đọc cần lưu ý đến những điều cơ bản dưới đây:

① Phân chia aggregate thật hợp lí dựa theo nghiệp vụ trong thực tế.

② Ghi rõ các thuộc tính của aggregate cũng như các objects con trong nó (có thể không cần ghi method cũng được).

③ Chú thích đầy đủ về các business logic trong sơ đồ (dưới dạng gạch đầu dòng).

Phần 1 đến đây là hết, hẹn gặp lại bạn đọc ở các bài viết tiếp theo trong series về DDD, xin cảm ơn.
