# Xây dựng Micro-service với gRPC (Phần 2)

Chào mừng bạn đọc đã quay trở lại với series về xây dựng micro-service bằng giao thức gRPC của tôi.

Bạn đọc có thể xem lại phần 1 [tại đây](https://viblo.asia/p/xay-dung-micro-service-voi-grpc-E1XVOZvXLMz)

Trong phần 2 này tôi xin phép được chia sẻ với bạn đọc những nội dung chính như sau:

1. Giới thiệu chung về project minh hoạ.
2. Chức năng của các modules trong project.
3. Luồng chạy chính.
4. Tổng kết.

Không chần chừ nữa, chúng ta hãy cùng nhau bắt đầu thôi.

## Giới thiệu chung về project

Project lần này tôi muốn giới thiệu tới bạn đọc đó là một API đơn giản với các chức năng chính như sau:

1. Signup, Login, Validate user
2. Người dùng của thể order sản phẩm

Rất đơn giản như vậy thôi nhưng khi triển khai bằng micro-service, tổng quan toàn bộ project trông sẽ như sau:

![Screenshot 2024-05-01 at 16 29 31](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/360b5ea2-878b-43c7-8460-08f58c082454)

Như các bạn có thể thấy, project sẽ gồm 4 modules chính đó là:

1. API-Gateway
2. Authen Service
3. Order Service
4. Item Service

Data model sẽ như sau:

![Screenshot 2024-05-01 at 18 19 04](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/c95ea438-8ae2-4999-a4fc-800ce9e4397a)

Do đây là micro-service nên mỗi một service sẽ có riêng cho mình một database, database này chỉ chứa các bảng liên quan đến nghiệp vụ chính của từng service mà thôi.

Giải thích sơ qua về data model như sau:

- Authen service do quản lí các tính năng liên quan đến xác thực "người dùng" nên tôi sẽ đưa bảng User vào database của Authen service.
- Order service do quản lí các tính năng liên quan đến order nên tôi sẽ đưa thông tin liên quan đến một lần order vào database của Order service.
- Item service sẽ quản lí các thông tin liên quan đến sản phẩm (item) nên database của nó sẽ chứa bảng Item.

Để hiểu rõ hơn về nhiệm vụ của từng modules, xin mời bạn đọc tiếp tục theo dõi các phần dưới đây.

## API-Gateway

Rất đơn giản thôi, module này hoạt động như một API-Gateway thông thường. Cho những bạn đọc nào còn chưa rõ về API-Gateway thì đây là một thành phần thường gặp trong các API, đúng như cái tên "Gateway" nó hoạt động giống như một cổng vào của hệ thống với chức năng như sau:

1. "Kết tập" các Backend-services.
2. Định nghĩa các endpoints mà hệ thống cho phép bên ngoài gọi vào.
3. Nhận request từ bên ngoài, chuyển tiếp request đến handler tương ứng với endpoint.
4. Trả về kết quả xử lí của từng endpoint.

Mô tả đơn giản bằng hình vẽ như sau:

![Screenshot 2024-05-01 at 17 41 08](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/3bbc986d-4d86-4d85-ad27-8505f8620ddf)

## Authen Service Module

Module này thực hiện 3 chức năng chính:

1. Signup
2. Login
3. Validate user

Bạn đọc có thể tham khảo source code tại đây: <https://github.com/tuananhhedspibk/micro-buying/tree/main/auth-svc>

## Order Service

Module này thực hiện 2 chức năng chính:

1. Tạo một order mới.
2. Lấy ra order dựa theo id.

Bạn đọc có thể tham khảo source code tại đây: <https://github.com/tuananhhedspibk/micro-buying/tree/main/order-svc>

## Item Service

Module này thực hiện 3 chức năng chính:

1. Tạo mới item.
2. Cập nhật thông tin item.
3. Lấy ra item dựa theo id.

Bạn đọc có thể tham khảo source code tại đây: <https://github.com/tuananhhedspibk/micro-buying/tree/main/item-svc>

## Luồng chạy chính

## Tổng kết
