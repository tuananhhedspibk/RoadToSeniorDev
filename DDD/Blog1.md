# Tôi đã xây dựng một API đơn giản bằng DDD như thế nào ?

## Tổng quan về API

Tôi dev API này với mục đích chính là để thực hành thêm về DDD nên sẽ không quá chú trọng vào các chức năng phức tạp. Về cơ bản API của tôi có thể được mô tả bằng sơ đồ usecase như sau:

![Screen Shot 2023-06-01 at 8 06 37](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/941b8d88-cdcd-46b5-918f-3fa7b8d022ce)

Về cơ bản đây là một API cho phép người sử dụng có thể:

- Tạo bài đăng với ảnh đi kèm
- Comment / Like bài đăng
- Follow / Unfollow những người dùng khác

Bạn đọc có thể tham khảo source code của tôi ở địa chỉ: <https://github.com/tuananhhedspibk/NewAnigram-BE-DDD>

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
