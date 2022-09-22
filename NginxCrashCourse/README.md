# Nginx Crash

## Nginx là gì ?

Nginx là server cung cấp nội dung cho trang web

![File_000 (1)](https://user-images.githubusercontent.com/15076665/191643940-dd7c86ff-3203-4816-9359-2c03237d6ed4.png)

Không những thế, trong trường hợp hệ thống có nhiều servers thì nginx còn hoạt động như một `load balancer` giúp điều phối các requests đến với các servers tương ứng.

Như hình minh hoạ ở trên ta thấy nginx có thể hoạt động như một `Reverse proxy`

Trong trường hợp hệ thống cần mã hoá (encryption), thay vì phải mã hoá và giải mã hoá ở từng servers, ta sẽ thiết lập mã hoá và giải mã hoá ở nginx server là đủ.

## Configuration Terms

Ta có 2 khái niệm config cơ bản trong nginx đó là:

- Context
- Directive

```nginx.conf
http { # context, tương đương như scope
  index index.html;  # directive: giống như một config

  server {
    list 80;
  }
}
```

Do `context` tương đương như scope giống như đoạn code ở trên nên các `context` có thể lồng nhau.

Có 3 `context` quan trọng của nginx cần chú ý đó là:

- http
- server
- location

```shell
nginx -t
```

Dùng để kiểm tra syntax của file `nginx.conf`

## Reload nginx

```shell
nginx -s reload
```

Sự khác biệt cơ bản giữa `nginx restart` và `nginx reload` đó là:

- `nginx reload` sẽ tắt tiến trình cũ của nginx đi, dựa theo config file để khởi tạo lại một tiến trình mới (lưu ý ở đây nginx service **KHÔNG BỊ TẮT**), trong trường hợp file config có vấn đề thì toàn bộ tiến trình sẽ bị dừng lại
- `nginx restart` sẽ tắt hẳn nginx đi và bật lại nginx service mới. Tuy nhiên nếu config file gặp vấn đề thì nginx service sẽ bị **DỪNG HẲN**

## Nginx location block

```nginx.conf
location URI {
  # handle response
}
```

Với `URI` ta cũng có thể sử dụng regex - `REGEX match` như ví dụ sau:

```nginx.conf
location ~ /count/[0-9] {
  root path;
}
```

`~` symbol cho thấy ta sẽ sử dụng regex ở đây, tuy nhiên mặc định thì đây là `case sensitive` (phân biệt hoa thường), còn nếu thêm `*` symbol thì sẽ là `case insensitive` (không phân biệt hoa thường) như ví dụ sau:

```nginx.conf
location ~ /test[0-9] {
  return 200 "test";
}

# Lúc này /test0 sẽ khác với /Test0

location ~* /test[0-9] {
  return 200 "test";
}

# Lúc này /test0 sẽ giống như /Test0
```

Cách viết `location URI` về bản chất là `prefix-match` nên nếu ta thiết lập `URI = test` thì các URIs khác ví dụ như:

- /testing
- /tested

cũng sẽ match. Do đó ta còn có cách viết `exact-match` như sau:

```nginx.conf
location = URI {
  # response
}
```

VD:

```nginx.conf
location = /test {
  return 200 "Hello test";
}
```

Xét về độ ưu tiên thì `REGEX-match` có độ ưu tiên cao hơn `prefix-match`. Tuy nhiên nếu ta sử dụng `^~` - `Preferential match` thì `prefix-match` sẽ có độ ưu tiên cao hơn `REGEX-match`

Có 3 cách viết với response như sau:

**Cách 1:**

```nginx.conf
location /test {
  # return HTTP_STATUS_CODE "response content"
  return 200 "Response content";
}
```

Ở cách viết trên, với trường hợp ta thiết lập `HTTP_STATUS_CODE = 200`, ta sẽ thu được response như sau:

![Screen Shot 2022-09-22 at 12 23 06](https://user-images.githubusercontent.com/15076665/191651475-e6534c53-44ee-4fc4-8d5e-9cc835a53069.png)

còn với thiết lập `HTTP_STATUS_CODE = 404`, ta sẽ thu được response như sau:

![Screen Shot 2022-09-22 at 12 23 17](https://user-images.githubusercontent.com/15076665/191651480-cd1197bb-34ab-45fb-8f83-2b2673a532fc.png)

**Cách 2:**

```nginx.conf
location /test {
  root path;
}
```

Với cách viết sử dụng `root` thì URI sẽ được tự động append vào path khi đó ta sẽ có `path/URI`

VD:

```nginx.conf
location /test {
  root /var/www/home;
}
```

Khi client truy cập vào URI `/test` thì req sẽ được redirect đến thư mục `/var/www/home` + `/test` = `/var/www/home/test`
Cách viết này sẽ "nói" cho nginx biết là hãy tìm đến file `index.html` (mặc định). Tuy nhiên trong trường hợp trong folder `/var/www/home/test` không có file `index.html` thì sao ?

Lúc này hãy sử dụng `try_files` như sau:

```nginx.conf
location /test {
  root /var/www/home;
  try_files test.html index.html =404;
}
```

Viết như trên có nghĩa là khi req đến `/test` hãy tìm đến file `test.html` trong folder `/var/www/home/test` nếu **không thấy** thì tìm đến file `index.html` trong `/var/www/home`, còn nếu trong trường hợp **cũng không thấy** `index.html` thì sẽ trả về `404 error NOT FOUND`

**Cách 3:**

```nginx.conf
location URI {
  alias path;
}
```

Cách viết này sử dụng từ khoá `alias`, alias **KHÔNG TỰ append** URI vào path cho ta như `root` thay vào đó ta phải chỉ định path một cách tường minh

```nginx.conf
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

```nginx.conf
location URI1 {
  return 307 REDIRECT_URI;
}
```

Ở đây `307` chính là HTTP Code của redirect request

**Rewrite:**

Trong trường hợp bạn không muốn bị redirect đến một trang khác, vẫn giữ nguyên URL như cũ nhưng chỉ khác nội dung thì `rewrite` là một giải pháp

VD:

```nginx.conf
rewrite ^/number/(\w+) /count/$1;
```

Ở ví dụ trên ta thấy `(\w+)` được bao bởi một cặp ngoặc đơn `()`, việc này giúp ta có thể sử dụng giá trị của nó ở `$1`. Ví dụ nếu URI là `/number/100` thì giá trị của `$1` sẽ là `100`

`rewrite` có mức độ ưu tiên cao hơn `location`

## Variables

Trong nginx có 2 loại biến chính:

- Configuration variables (đây là các biến mà ta sẽ tự định nghĩa): `set $var 'something';`
- NGINX Module variables (đây là các biến có sẵn, do nginx module định nghĩa): `$http`, `$uri`, `$args` - query string params
