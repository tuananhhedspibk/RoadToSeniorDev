# Ghi chép nhanh về gRPC

Thao khảo từ:

1. <https://200lab.io/blog/grpc-la-gi/>
2. <https://medium.com/javarevisited/what-is-grpc-is-it-better-than-rest-api-58a3b7aff13a>

## RPC là gì

RPC là tên viết tắt của "Remote Procedure Call" - lời gọi thủ tục từ xa. Có thể hiểu nó giống như việc **gọi 1 hàm của server từ phía client**.

RPC chỉ sử dụng POST và GET method. Tương tác giữa client và server sẽ thông qua `Server-side Interface` và `Client-side stub`

## Khác biệt giữa RPC và REST

REST được thiết kế hướng "tài nguyên" - tức là mọi thao tác của REST (thêm mới, sửa, xoá) đều tác động lên tài nguyên (resource).

Và đương nhiên kết quả mà REST sẽ trả về đó là "resource"

Còn RPC sẽ chỉ thực hiện việc gọi một hàm với 1 chức năng nhất định của phía server và dữ liệu RPC trả về chỉ tuần tuý là giá trị trả về của hàm phía server.

Lấy ví dụ:

- Với REST ta có:

1. GET `/posts/1`
2. POST `/posts/1`
3. PUT `/posts/1`

- Với RPC ta có:

1. `/posts/1/calculate_score`
2. `/posts/1/average_score`

Trong thực tế thì RPC sẽ được triển khai dựa trên REST nên ta có khái niệm **RPC-base APIs**.

![Screen Shot 2023-09-23 at 16 02 24](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/ac2567e6-f17c-4de4-986a-c4c07855a780)

## gRPC là gì? Nó có gì khác biệt

> gRPC là một open source remote procedure call (RPC) framework có thể chạy ở bất kì đâu. Nó cho phép client và server app có thể tương tác một cách "minh bạch".

gRPC là một framework được phát triển bởi Google. Giống như các hệ thống RPCm gRPC cũng định nghĩa interface - bản thân interface này cũng định nghĩa các methods mà client có thể truy cập.

Mặc định gRPC sử dụng protocol buffers như một `Interface Definition Language` & data exchange format. gRPC hỗ trợ các format như JSON.

File `Interface Defintiion Language` trong gRPC được định nghĩa bởi file có extension là `.prot`.

> Chúng ta hoàn toàn có thể quản lí file này dựa theo version.

Các loại RPC mà gRPC hỗ trợ gồm:

1. **Unary**: Single request từ client - Single response từ server.
2. **Server streaming**: Single request từ client, stream messages trả về từ server.
3. **Client Streaming**: Stream messages từ client và trả về single response từ server.
4. **Bidirectional streaming**: Cả client và server đều gửi stream messages.

## gRPC được áp dụng trong thực tế như thế nào

Với hệ thống microservices với nhiều service liên lạc với nhau, việc sử dụng REST sẽ phát sinh một vấn đề đó là encoding/ decoding dữ liệu (JSON data), còn RPC thì không cần phải encoding hay decoding dữ liệu.

Do đó gRPC thường được sử dụng trong các hệ thống microservice cần chịu tải lớn.

Hơn nữa gRPC được xây dựng trên nền tảng HTTP/2 hỗ trợ stream nên nó sẽ thích hợp cho việc stream event trong event sourcing, việc HTTP/2 sử dụng binary data thay vì text cũng sẽ làm tăng tốc độ cho việc liên lạc giữa các services với nhau.

HTTP/2 sử dụng một kiểu dữ liệu do Google phát minh đó là `Protobuf - Protocol Buffer`, tốc độ encoding/ decoding của nó khi so với text hay JSON sẽ như sau:

![Screen Shot 2023-09-23 at 16 07 14](https://github.com/tuananhhedspibk/DDD-Modeling/assets/15076665/2600ac80-975d-47f4-9fd0-626dc07f62e1)

Một chú ý khác đó là gRPC nên dùng cho giao tiếp giữa `backend - backend` thay vì `backend-frontend` do việc giao tiếp stateful giữa `backend-frontend` sẽ gây ra các vấn đề "scale tải" hoặc "HOL"
