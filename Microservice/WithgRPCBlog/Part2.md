# Xây dựng Micro-service với gRPC (Phần 2)

Chào mừng bạn đọc đã quay trở lại với series về xây dựng micro-service bằng giao thức gRPC của tôi.

Bạn đọc có thể xem lại phần 1 [tại đây](https://viblo.asia/p/xay-dung-micro-service-voi-grpc-E1XVOZvXLMz)

Trong phần 2 này tôi xin phép được chia sẻ với bạn đọc những nội dung chính như sau:

1. Giới thiệu chung về project minh hoạ.
2. Chức năng của các modules trong project.
3. Luồng chạy chính.
4. Tổng kết.

Không chần chừ nữa, chúng ta hãy cùng nhau bắt đầu thôi.

## Giới thiệu chung về project minh hoạ

Project lần này tôi muốn giới thiệu tới bạn đọc đó là một API đơn giản với các chức năng chính như sau:

1. Signup, Login, Validate user.
2. User có thể order sản phẩm.
3. User có thể lấy về thông tin của một order.

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
3. Nhận request từ bên ngoài, chuyển tiếp request đến handler tương ứng với từng endpoint.
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
2. Lấy ra thông tin của order dựa theo id.

Bạn đọc có thể tham khảo source code tại đây: <https://github.com/tuananhhedspibk/micro-buying/tree/main/order-svc>

## Item Service

Module này thực hiện 3 chức năng chính:

1. Tạo mới item.
2. Cập nhật thông tin item.
3. Lấy ra thông tin của item dựa theo id.

Bạn đọc có thể tham khảo source code tại đây: <https://github.com/tuananhhedspibk/micro-buying/tree/main/item-svc>

## Luồng chạy chính

![Screenshot 2024-05-04 at 11 14 03](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/71d16469-2aa1-4bd5-8448-d79b0b980231)

Cùng phân tích hình vẽ trên, chúng ta lấy 2 ví dụ tương ứng với 2 luồng chức năng:

1. Đăng kí (đi theo đường mũi tên màu xanh nước biển).
2. Order sản phẩm (đi theo đường mũi tên màu xanh lá, mũi tên màu cam sẽ là riêng của item-service).

### Đăng kí

Các requests gửi từ phía client đến cho API-Gateway sẽ đều là HTTP request. API-Gateway sau khi nhận request từ client sẽ thực hiện lời gọi RPC đến cho `Auth-Service` kèm theo các params cần thiết. `Auth-Service` nhận lời gọi RPC, thực hiện các xử lí cần thiết và ghi kết quả vào `User-DB`.

Cụ thể hơn như thế nào tôi xin phép được giới thiệu tới bạn đọc ở các phần tiếp theo.

### Order sản phẩm

Cũng tương tự như chức năng đăng kí, client sẽ gửi HTTP request order sản phẩm đến cho API-Gateway. API-Gateway sau đó sẽ chuyển từ lời gọi HTTP sang lời gọi RPC và chuyển tiếp nó đến cho `Order-Service`. `Order-Service` sẽ tiến hành các xử lí và lưu kết quả vào `Order-DB`, thế nhưng ở đây có một sự khác biệt "nho nhỏ" ở chỗ do order sẽ luôn đi kèm với sản phẩm nên sẽ có sự tương tác giữa `Order-Service` và `Item-Service` cụ thể là khi tiến hành order sản phẩm ta cần cập nhật trạng thái của sản phẩm đó từ "SẴN CÓ" thành "ĐÃ BÁN".

Sự tương tác giữa `Order-Service` và `Item-Service` sẽ được thực thi bằng event-based - cụ thể hơn là SAGA Pattern. Các bạn có thể tham khảo bài viết dưới đây của tôi để có một cái nhìn sơ lược nhất về SAGA Pattern.

<https://viblo.asia/p/tuong-tac-giua-cac-micro-services-bang-aws-ecs-run-task-bXP4WMMkL7G>

Trở lại với luồng order sản phẩm, sau khi nhận được lệnh trigger từ phía `Order-Service`, `Item-Service` sẽ thực hiện các xử lí cần thiết để cập nhật trạng thái của sản phẩm từ "SẴN CÓ" thành "ĐÃ BÁN" và sau đó cập nhật lại vào `Item-DB` (luồng mũi tên màu cam).

## Tổng kết

Tóm lại trong phần này tôi đã trình bày với bạn đọc một cách sơ lược nhất về project demo của chúng ta hi vọng đây sẽ là tiền đề để bạn đọc có thể tiếp tục theo dõi các bài viết tiếp theo trong series về "Xây dựng Micro-service với gRPC" của tôi. Xin cảm ơn và hẹn gặp lại.
