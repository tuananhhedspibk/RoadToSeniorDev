# Sử dụng ECS Run Task cho mục đích tương tác giữa các micro-services

## Tương tác giữa các micro-services

Trong hệ thống gồm nhiều services, việc tương tác giữa các services là điều thường xuyên diễn ra. Các tương tác này có thể thông qua:

- API calling: đây là cách gọi API sử dụng HTTP hoặc RPC
- Event trigger: có thể hiểu là việc service A sẽ "gọi" đến service B thông qua việc emit một hoặc một vài events nào đó.

Về cách sử dụng API calling - đây là cách làm khá phổ biến nên tôi sẽ không đề cập đến trong bài viết lần này, thay vào đó tôi muốn đi sâu vào cách thứ hai, đó là sử dụng "Event trigger".

## SAGA pattern

Do mỗi một service sẽ có cho mình một DB riêng, nên việc service A "gọi" service B dẫn đến việc ta phải thực thi các transactions trên các DB khác nhau (chú ý rằng ta KHÔNG THỂ triển khai ACID transaction trên nhiều DB khác nhau).

Dẫn đến cần có một giải pháp để đảm bảo sử "thống nhất" giữa "các DB" với nhau.

Giải pháp ở đây đó chính là SAGA, có thể hiểu đơn giản rằng SAGA là một chuỗi các local transaction (transaction thuộc về một DB).

> Mỗi một local transaction sẽ update DB mà nó thuộc về, đống thời publish message hoặc event để trigger local transaction của service kế tiếp trong chuỗi logic nghiệp vụ

Như hình minh hoạ dưới đây.

![Screen Shot 2023-12-11 at 23 33 51](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/1385f989-20dc-4afb-b732-c121b8b6bf72)
