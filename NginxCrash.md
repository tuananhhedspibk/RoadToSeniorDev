# Nginx Crash

## Nginx là gì ?

Nginx là server cung cấp nội dung cho trang web, không những thế nginx cũng có thể được sử dụng như `reverse proxy`, `load balancer`, `HTTP cache` như hình minh hoạ dưới đây

![File_000 (1)](https://user-images.githubusercontent.com/15076665/191643940-dd7c86ff-3203-4816-9359-2c03237d6ed4.png)

Không những thế, trong trường hợp hệ thống có nhiều servers thì nginx còn hoạt động như một `load balancer` giúp điều phối các requests đến với các servers tương ứng.

Như hình minh hoạ ở trên bạn thấy nginx có thể hoạt động như một `Reverse proxy`

Trong trường hợp hệ thống cần mã hoá (encryption), thay vì phải mã hoá và giải mã hoá ở từng servers, bạn sẽ thiết lập mã hoá và giải mã hoá ở nginx server là đủ.

Phần cài đặt nginx các bạn có thể tham khảo tại link <https://www.nginx.com/resources/wiki/start/topics/tutorials/install/>

## Configuration Terms

Có 2 khái niệm config cơ bản trong nginx đó là:

- Context
- Directive

```nginx
http { # context, tương đương như scope
  index index.html;  # directive: giống như một config

  server {
    list 80;
  }
}
```

Do `context` tương đương như scope, giống như đoạn code ở trên các `contexts` có thể lồng nhau.

Có `3 contexts` quan trọng của nginx cần chú ý đó là:

- http
- server
- location

Các config trên thường sẽ được đặt trong file config của nginx với tên là `nginx.conf`, mặc định thì `nginx service` của hệ thống sẽ sử dụng file `/usr/local/etc/nginx/nginx.conf` (đây là đường dẫn trên hệ thống của tôi với hệ điều hành macOS, các bạn với hệ điều hành Linux hay Window sẽ có sự khác biệt).

Tuy nhiên bạn vẫn có thể sử dụng file config nginx của cá nhân mình bằng việc sử dụng câu lệnh như dưới đây

```shell
nginx -c file_path
```

## Reload nginx

```shell
nginx -s reload
```

Sự khác biệt cơ bản giữa `nginx restart` và `nginx reload` đó là:

- `nginx reload` sẽ tắt tiến trình cũ của nginx đi, dựa theo config file để khởi tạo lại một tiến trình mới (lưu ý ở đây nginx service **KHÔNG BỊ TẮT**), trong trường hợp file config có vấn đề thì toàn bộ tiến trình sẽ bị dừng lại
- `nginx restart` sẽ tắt hẳn nginx đi và bật lại nginx service mới. Tuy nhiên nếu config file gặp vấn đề thì nginx service sẽ bị **DỪNG HẲN**

## Nginx location block

```nginx
location URI {
  # handle response
}
```

Có 3 cách config cho URI:

- Regex match
- Prefix match
- Exact match

Với `Regex match` bạn có thể làm như ví dụ sau:

```nginx
location ~ /count/[0-9] {
  root path;
}
```

`~` symbol cho thấy bạn sẽ sử dụng regex ở đây, tuy nhiên mặc định thì đây là `case sensitive` (phân biệt hoa thường), còn nếu thêm `*` symbol thì sẽ là `case insensitive` (không phân biệt hoa thường) như ví dụ sau:

```nginx
location ~ /test[0-9] {
  return 200 "test";
}

# Lúc này /test0 sẽ khác với /Test0

location ~* /test[0-9] {
  return 200 "test";
}

# Lúc này /test0 sẽ giống như /Test0
```

Với `Prefix match` nếu bạn thiết lập `URI = test` thì các URIs khác ví dụ như:

- /testing
- /tested

cũng sẽ match.

Còn với `Exact match` bạn thêm dấu `=` phía trước URI như sau:

```nginx
location = URI {
  # response
}
```

VD:

```nginx
location = /test {
  return 200 "Hello test";
}
```

Xét về độ ưu tiên thì `Regex match` có độ ưu tiên cao hơn `Prefix match`.
Tuy nhiên nếu bạn sử dụng `^~` - `Preferential match` thì `Prefix match` lúc này sẽ có độ ưu tiên cao hơn `Regex match`

Với response bạn có 3 cách viết như sau:

**Cách 1 (sử dụng return):**

```nginx
location /test {
  # return HTTP_STATUS_CODE "response content"
  return 200 "Response content";
}
```

Ở cách viết trên, với trường hợp bạn thiết lập `HTTP_STATUS_CODE = 200`, bạn sẽ thu được response như sau:

![Screen Shot 2022-09-22 at 12 23 06](https://user-images.githubusercontent.com/15076665/191651475-e6534c53-44ee-4fc4-8d5e-9cc835a53069.png)

còn với thiết lập `HTTP_STATUS_CODE = 404`, bạn sẽ thu được response như sau:

![Screen Shot 2022-09-22 at 12 23 17](https://user-images.githubusercontent.com/15076665/191651480-cd1197bb-34ab-45fb-8f83-2b2673a532fc.png)

**Cách 2 (sử dụng root):**

```nginx
location /test {
  root path;
}
```

Với cách viết sử dụng `root` thì URI sẽ được tự động append vào path khi đó bạn sẽ có `path/URI`

VD:

```nginx
location /test {
  root /var/www/home;
}
```

Khi client truy cập vào URI `/test` thì request sẽ được redirect đến thư mục `/var/www/home` + `/test` = `/var/www/home/test`

Cách viết này sẽ "nói" cho nginx biết là hãy tìm đến file `index.html` (mặc định). Tuy nhiên trong trường hợp trong folder `/var/www/home/test` không có file `index.html` thì sao ?

Lúc này hãy sử dụng `try_files` như sau:

```nginx
location /test {
  root /var/www/home;
  try_files test.html index.html =404;
}
```

Viết như trên có nghĩa là khi request đến `/test` hãy tìm đến file `test.html` trong folder `/var/www/home/test` nếu **không thấy** thì tìm đến file `index.html` trong `/var/www/home`, và nếu như trong trường hợp **cũng không thấy** `index.html` thì sẽ trả về `404 error NOT FOUND`

**Cách 3 (sử dụng alias):**

```nginx
location URI {
  alias path;
}
```

Cách viết này sử dụng từ khoá `alias`, alias **KHÔNG TỰ append** URI vào path cho bạn như `root` thay vào đó bạn phải chỉ định path một cách tường minh

```nginx
location /test {
  root /var/www/home;
}

location /bruh {
  alias /var/www/home/test;
}
```

Với config như trên thì khi truy cập vào 2 URI là `/test` và `/bruh` thì request đều sẽ được redirect đến folder `/var/www/home/test`

## Redirect & Rewrite

**Redirect:**

```nginx
location URI1 {
  return 307 REDIRECT_URI;
}
```

Ở đây `307` chính là HTTP Code của redirect request

**Rewrite:**

Trong trường hợp bạn không muốn bị redirect đến một trang khác, vẫn giữ nguyên URL như cũ nhưng chỉ khác nội dung thì `rewrite` là một giải pháp

VD:

```nginx
rewrite ^/number/(\w+) /count/$1;
```

Ở ví dụ trên bạn thấy `(\w+)` được bao bởi một cặp ngoặc đơn `()`, việc này giúp bạn có thể sử dụng giá trị của nó ở `$1`. Ví dụ nếu URI là `/number/100` thì giá trị của `$1` sẽ là `100`

Chú ý là `rewrite` có mức độ ưu tiên cao hơn `location`

## Variables

Trong nginx có 2 loại biến chính:

- Configuration variables (đây là các biến mà bạn sẽ tự định nghĩa): `set $var 'something';`
- NGINX Module variables (đây là các biến có sẵn, do nginx module định nghĩa): `$http`, `$uri`, `$args` - query string params

## Logging

nginx có `access log` và `error log` lần lượt lưu log của những request kết nối tới và log lỗi

```shell
nginx -V
```

Chạy câu lệnh trên bạn có thể thấy được đường dẫn đến 2 file `access.log` và `error.log` mặc định của nginx trên hệ thống

![Screen Shot 2022-09-22 at 17 40 57](https://user-images.githubusercontent.com/15076665/191700832-2397836a-1a19-4898-8ade-080418b0f78f.png)

```nginx
location URI {
  access_log log_file_path;
}
```

Bằng cách thiết lập như trên chúng bạn có thể chỉ định ra file chứa access log riêng biệt so với file access.log mặc định của nginx

## Tham khảo

<https://www.youtube.com/watch?v=7VAI73roXaY>
<https://www.udemy.com/course/nginx-fundamentals/>
