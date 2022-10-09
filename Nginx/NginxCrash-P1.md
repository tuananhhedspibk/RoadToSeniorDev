# nginx Crash - Part 1

nginx là một công cụ rất quen thuộc với những người làm server (backend) nói chung, việc nắm được những concepts cơ bản về nó là một điều rất cần thiết. Trong bài viết lần này tôi sẽ chia sẻ cho bạn đọc những ghi chép cá nhân của tôi trong quá trình tìm hiểu về nginx.

Hai nguồn chính tôi tham khảo và học hỏi ở đây là:

- <https://www.youtube.com/watch?v=7VAI73roXaY>
- <https://www.udemy.com/course/nginx-fundamentals/>

## nginx là gì ?

nginx là server cung cấp nội dung cho trang web, không những thế nginx cũng có thể được sử dụng như `reverse proxy`, `HTTP cache` như hình minh hoạ dưới đây

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
  server {
    listen 8080;
    root path_to_mysite/; # directive: giống như một config
  }
}
```

Do `context` tương đương như scope, giống như đoạn code ở trên các `contexts` có thể lồng nhau.

Có `3 contexts` quan trọng của nginx cần chú ý đó là:

- http
- server
- location

Các config thường sẽ được đặt trong file config của nginx với tên là `nginx.conf`, mặc định thì `nginx service` của hệ thống sẽ sử dụng file `/usr/local/etc/nginx/nginx.conf` (đây là đường dẫn trên hệ thống của tôi với hệ điều hành macOS, các bạn với hệ điều hành Linux hay Window sẽ có sự khác biệt).

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
- `nginx restart` sẽ **TẮT HẲN** nginx đi và **BẬT LẠI** nginx service mới. Tuy nhiên nếu config file gặp vấn đề thì nginx service sẽ bị **DỪNG HẲN**

## Serve static content

Với các static content như HTML hay CSS, bạn cần thêm định nghĩa về types như sau:

```nginx
http {
  types {
    text/html html;
    text/css css;
  }
}
```

Các types này có thể được tham khảo ở file `mime.types` trong hệ thống của bạn (với tôi đó là file `/usr/local/etc/nginx/mime.types`)

Lấy ví dụ với `css`, nếu ta không có directive `text/css css;` thì khi load file `.css` về thì trình duyệt sẽ hiểu nó là file `text/plain` như hình bên dưới, lúc này các style bên trong file `css` sẽ không được apply cho trang HTML của bạn.

![Screen Shot 2022-09-23 at 12 09 18](https://user-images.githubusercontent.com/15076665/191885536-d2e88c1a-0088-4d5a-a005-b7be6ea9da31.png)

Nếu có directive `text/css css;` thì trình duyệt sẽ hiểu đó là file `text/css`, khi đó các style sẽ được áp dụng cho trang HTML của bạn.

![Screen Shot 2022-09-23 at 12 13 18](https://user-images.githubusercontent.com/15076665/191885752-902410e7-c25d-4221-ae64-ce539f3ced6c.png)

## nginx location block

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
# Regex match - case sensitive
location ~ /test[0-9] {
  return 200 "Hello from test";
}

# Lúc này /test0 sẽ khác với /Test0

# Regex match - case insensitive
location ~* /test/[0-9] {
  return 200 "Hello from haha test";
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
  return 200 "Hello from test";
}
```

Lúc này config chỉ matching URI = "/test" mà thôi, các URIs như "/testing", "/tested", ... sẽ không được matching.

Xét về độ ưu tiên thì `Regex match` có độ ưu tiên cao hơn `Prefix match`.
Tuy nhiên nếu bạn sử dụng `^~` - `Preferential match` thì `Prefix match` lúc này sẽ có độ ưu tiên cao hơn `Regex match`

```nginx
# Preferential match
location ^~ /test1 {
  return 200 "Hello from test";
}
```

Với response bạn có 3 cách viết như sau:

**Cách 1 (sử dụng return):**

```nginx
location /test {
  # return HTTP_STATUS_CODE "response content"
  return 200 "Response content";
}
```

Ở cách viết trên, với trường hợp bạn thiết lập `HTTP_STATUS_CODE = 200`

```nginx
location /test {
  return 200 "Response content";
}
```

bạn sẽ thu được response như sau:

![Screen Shot 2022-09-22 at 12 23 06](https://user-images.githubusercontent.com/15076665/191651475-e6534c53-44ee-4fc4-8d5e-9cc835a53069.png)

còn với thiết lập `HTTP_STATUS_CODE = 404`

```nginx
location /not-found {
  return 404 "Response not found";
}
```

bạn sẽ thu được response như sau:

![Screen Shot 2022-09-22 at 12 23 17](https://user-images.githubusercontent.com/15076665/191651480-cd1197bb-34ab-45fb-8f83-2b2673a532fc.png)

**Cách 2 (sử dụng root):**

```nginx
location /URI {
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

Khi client truy cập vào URI `/test` thì request sẽ được chuyển đến thư mục `/var/www/home` + `/test` = `/var/www/home/test`

Cách viết này sẽ "nói" cho nginx biết là hãy tìm đến file `index.html` (mặc định). Tuy nhiên trong trường hợp trong folder `/var/www/home/test` không có file `index.html` thì sao ?

Lúc này hãy sử dụng `try_files` như sau:

```nginx
location /vegetables {
  root path_to_mysite/;
  try_files /vegetables/veggies.html /index.html =404;
}
```

Viết như trên có nghĩa là khi request đến `/vegetables` hãy tìm đến file `/vegetables/veggies.html` trong folder `path_to_mysite/vegetables` nếu **không thấy** thì tìm đến file `index.html` trong `path_to_mysite/`, và nếu như trong trường hợp **cũng không thấy** `index.html` thì sẽ trả về `404 NOT FOUND` error

**Cách 3 (sử dụng alias):**

```nginx
location URI {
  alias path;
}
```

Cách viết này sử dụng từ khoá `alias`, alias **KHÔNG TỰ APPEND** URI vào path cho bạn như `root` thay vào đó bạn phải chỉ định path một cách tường minh. Ví dụ:

```nginx
location /fruits {
  root path_to_mysite/;
}

location /carbs {
  alias path_to_mysite/fruits;
}
```

Với config như trên thì khi truy cập vào 2 URIs là `/fruits` và `/carbs` thì request đều sẽ được chuyển đến folder `path_to_mysite/fruits`

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

> Chú ý là rewrite có mức độ ưu tiên cao hơn location

## Variables

Trong nginx có 2 loại biến chính:

- Configuration variables (đây là các biến mà bạn sẽ tự định nghĩa): `set $var 'something';`
- NGINX Module variables (đây là các biến có sẵn, do nginx module định nghĩa): `$http`, `$uri`, `$args` - query string params

```nginx
set $isthursday 'No';

if ( $date_local ~ 'Thursday' ) {
  set $isthursday 'Yes';
}

location /inspect {
  return 200 "Is Thursday: $isthursday";
}
```

Ở ví dụ trên ta kiểm tra biến định nghĩa sẵn của nginx là `$date_local` - ngày hiện tại xem có phải là thứ năm (Thursday) hay không ? Nếu đúng thì gán cho biến `$isthursday` giá trị là `Yes` (biến này ban đầu được thiết lập giá trị mặc định là `No`)

Trong nginx, bạn tiến hành "nhúng" giá trị của biến vào một string bằng cách thêm kí tự `$` phía trước tên biến như ở ví dụ trên với biến `isthursday`

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

Bằng cách thiết lập như trên bạn có thể chỉ định ra file chứa access log riêng biệt so với file access.log mặc định của nginx

## Reverse proxy

Khái niệm về `Reverse proxy` các bạn có thể tham khảo ở nguồn <https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/>

Hiểu đơn giản thì `Reverse proxy` sẽ được đặt ở phía trước server, client thay vì tương tác trực tiếp với server, nó sẽ tương tác với `reverse proxy`. Mọi request từ client hoặc response từ server đều phải thông qua `reverse proxy`

Bạn có thể thiết lập reverse proxy đơn giản như sau:

```nginx
location /express {
  proxy_pass 'http://localhost:7777/';
}
```

Thiết lập này có nghĩa là khi gửi request đến `/express` thì request sẽ được chuyển tiếp đến địa chỉ `http://localhost:7777/`

Trong đường dẫn của `proxy_pass` nếu ta bỏ `/` phía sau `localhost:7777` đi thì nginx sẽ tự động redirect đến `localhost:7777/express`

Với headers thì ta có 2 directives cơ bản như sau `add_header`, `proxy_set_header`

```nginx
location /express {
  add_header proxied nginx;
  proxy_set_header custom myown;
  proxy_pass 'http://localhost:7777/';
}
```

`add_header` sẽ thêm vào header của response trả về từ phía proxy

![Screen Shot 2022-09-23 at 11 02 10](https://user-images.githubusercontent.com/15076665/191880224-27e12cca-d56c-4c54-8b5e-6f4c2dcaf8a3.png)

`proxy_set_header` sẽ thêm vào header của request gửi lên server từ phía proxy, nên trên trình duyệt ta không thể thấy được header này. Nguyên nhân là bởi ở trình duyệt bạn chỉ đang **TƯƠNG TÁC TRỰC TIẾP VỚI PROXY** chứ không phải tương tác với server

![Screen Shot 2022-09-23 at 11 02 22](https://user-images.githubusercontent.com/15076665/191880112-0357e686-25e8-4504-9337-faf307941dbb.png)

Hình dưới mô tả về "vị trí" thêm header của `add_header` và `proxy_set_header`

![File_000 (2)](https://user-images.githubusercontent.com/15076665/191886263-4e2bffa8-10d4-4171-a6de-7d4713ebfb38.png)

Muốn thấy được thì bạn có thể log trực tiếp header trong server ra console của mình như sau:

```js
app.get('/', (req, res) => {
  console.log("req.headers", req.headers);
  console.log("req.url", req.url);
  console.log("req.route", req.route);
  res.send('I am an endpoint');
});
```

Ở trên tôi sử dụng `expressJS` để làm server minh hoạ, các bạn có thể dùng bất kì server framework nào tuỳ ý.

Và đây là kết quả khi log ra console

![Screen Shot 2022-09-23 at 10 59 03](https://user-images.githubusercontent.com/15076665/191880155-7ccf5bfe-d826-4e10-8edf-04a029100ef9.png)

## Code tham khảo

- Phần location, redirect, rewrite, variable: <https://github.com/tuananhhedspibk/BlogCode/tree/main/NginxCrash/basic>
- Phần reverse-proxy: <https://github.com/tuananhhedspibk/BlogCode/tree/main/NginxCrash/reverse-proxy>

## Kết

Trên đây là một vài ghi chép của tôi trong quá trình tự tìm hiểu về nginx. Cảm ơn các bạn đã kiên nhẫn đọc hết, hẹn gặp lại ở phần 2 (to be continued).
