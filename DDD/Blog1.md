# Tôi đã xây dựng một API đơn giản bằng DDD như thế nào ?

## Tổng quan về API

Tôi dev API này với mục đích chính là để thực hành thêm về DDD nên sẽ không quá chú trọng vào các chức năng phức tạp. Về cơ bản API của tôi có thể được mô tả bằng sơ đồ usecase như sau:

![Screen Shot 2023-05-31 at 22 24 29](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/assets/15076665/09dd839a-6ed9-490d-9c24-7b7f7ee50a82)

Về cơ bản đây là một API cho phép người sử dụng có thể:

- Tạo bài đăng với ảnh đi kèm
- Comment / Like bài đăng
- Follow / Unfollow những người dùng khác

## DDD là gì ?

DDD (Domain Driven Design) là một cách thiết kế phần mềm "hướng nghiệp vụ" - tức sẽ lấy nghiệp vụ (domain) làm trung tâm của hệ thống.

Có thể ví dụ với hệ thống "nghiệp vụ ngân hàng" hoặc "nghiệp vụ kho vận". Nếu chỉ nói về mặt khái niệm thì sẽ rất khó hình dung DDD là gì nên do đó trong bài viết này tôi sẽ cố gắng minh hoạ từng khái niệm của DDD tương ứng với code API của mình.

### Một vài khái niệm cơ bản trong DDD

Vì là "domain" driven design nên **domain** là khái niệm chính ở đây. Xoay quanh khái niệm về domain ta có 3 key words chính đó là **Domain**, **Model** và **Domain Model**

**Domain** là một lĩnh vực nghiệp vụ nào đó ví dụ như "nghiệp vụ ngân hàng" hoặc "nghiệp vụ kho vận", ...

**Model** là quá trình trừu tượng hoá các đối tượng cũng như các thao tác của nghiệp vụ. Quá trình mô hình hoá nghiệp vụ thành các đối tượng mang tính trừu tượng này được gọi là quá trình **Modeling** hoặc **Mô hình hoá**.

**Domain Model** là các models ta thu được sau quá trình **Modeling** ở trên.
