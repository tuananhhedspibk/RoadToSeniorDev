# SSL hoạt động như thế nào

## SSL là gì ?

SSL là một phương thức mã hoá dữ liệu. Thường dùng để mã hoá dữ liệu giữa bên gửi và bên nhận.

TLS là phiên bản mới nhất của SSL nhưng vì thói quen, mọi người vẫn hay gọi là SSL

## Phương thức mã hoá

SSL sử dụng hai phương thức mã hoá là `Asymmetric Cryptography` và `Symmetric Cryptography`

`Asymmetric Cryptography` sử dụng một cặp key là `public key` và `private key`

`Symmetric Cryptography` chỉ sử dụng một key duy nhất và đó cũng là `private key`

## Xử lí dữ liệu

Quá trình truyền dữ liệu bằng SSL sẽ được tiến hành bằng 2 bước chính như sau:

① Handshake
② Data trasfer

Hai quá trình này được mô tả như hình phía dưới:

![File_000](https://user-images.githubusercontent.com/15076665/199931818-8860bec6-65e0-43a1-b800-1abaf2e678fa.png)

## SSL certificate

Được phát hành bởi CA, certificate này sẽ bao gồm:

- Public key
- Các thông tin chi tiết khác

Web server gửi `public key` cho client thông qua certificate
