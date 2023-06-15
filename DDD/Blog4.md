# Tôi đã xây dựng một API đơn giản bằng DDD như thế nào ? - Phần 4

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
