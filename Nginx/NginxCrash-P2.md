# nginx Crash - Part 2

Trong phần hai này tôi sẽ tiếp tục trình bày về những gì mình tự học được về nginx

Tư liệu tham khảo chính:

- <https://www.udemy.com/course/nginx-fundamentals/>

## Load Balancer

nginx có thể được sử dụng như một load balancer với hai nhiệm vụ như sau:

**Điều phối requests tới các servers:**

![File_000](https://user-images.githubusercontent.com/15076665/191892736-02fdf66b-eacc-4a9d-8696-a724281e2c6c.png)

**Phân tải khi có server gặp sự cố:**

![File_000 (1)](https://user-images.githubusercontent.com/15076665/191892743-3ac40b26-ce69-47c8-984a-cf86d544475c.png)

Như ở hình trên bạn thấy, khi server 1 gặp sự cố thì toàn bộ các request từ phía client sẽ được chuyển tiếp tới cho 2 server còn lại là `server 2` và `server 3`.

Các bạn có thể tham khảo code demo của tôi ở địa chỉ <https://github.com/tuananhhedspibk/BlogCode/tree/main/NginxCrash/load-balancer>

Phân tích một chút về môi trường. Ở đây tôi dựng nên 3 servers bằng `express JS` lần lượt lắng nghe ở các cổng `1111`, `2222`, `3333`.

**Server 1:**

```JS
app.listen(1111, () => {
  console.log('Listening on port 1111');
});
```

**Server 2:**

```JS
app.listen(2222, () => {
  console.log('Listening on port 2222');
});
```

**Server 3:**

```JS
app.listen(3333, () => {
  console.log('Listening on port 3333');
});
```

Load balancer sẽ lắng nghe ở cổng `8888`

```nginx
events {}

http {
  upstream express_servers {
    server localhost:1111;
    server localhost:2222;
    server localhost:3333;
  }

  server {
    listen 8888;

    location / {
      proxy_pass http://express_servers;
    }
  }
}
```

Ở phần config cho nginx trên, tôi sử dụng `upstream` (<http://nginx.org/en/docs/http/ngx_http_upstream_module.html>), context này giúp định nghĩa một nhóm các servers sẽ được tham chiếu sau đó bởi proxy, ...

Ở phần config cho proxy (lưu ý ở đây là cách config load balancer và proxy không khác nhau quá nhiều) tôi thêm directive
`proxy_pass http://express_servers;` có nghĩa là các req đến địa chỉ `localhost:8888/` sẽ được chuyển tiếp đến `stream server express_servers` như định nghĩa ở trên.

Ở đây các requests sẽ lần lượt được luân chuyển đến các servers `localhost:1111`, `localhost:2222`, `localhost:3333` được chạy bằng express ở trên. Thuật toán luân chuyển request mặc định sử dụng ở đây là `Round robin`.

Nói sơ qua thì đây là thuật toán luân chuyển requests đến các servers dựa theo trọng số (server weight) tương ứng với từng server. Weight càng cao thì tần suất nhận được request càng cao và ngược lại.
Mặc định thì weight của các servers là bằng nhau.

Để "xác minh" xem `load balancer` có hoạt động như ý muốn của chúng ta hãy không bạn hãy thử chạy lệnh sau trong terminal của mình

```shell
while sleep 1; do curl http://localhost:8888; done
```

Câu lệnh trên sẽ đều đặn thực hiện lệnh `curl http://localhost:8888` đến proxy của chúng ta cứ sau một giây một.
Và đây là kết quả chúng ta nhận được

![Screen Shot 2022-09-23 at 16 21 25](https://user-images.githubusercontent.com/15076665/191910278-62a2fe09-8940-49d1-9d0e-c660d375e2d0.png)

Các requests lần lượt được luân chuyển đến các server theo thứ tự 1, 2, 3

## Load balancer options

### Sticky sessions (ip hash)

Dựa theo địa chỉ IP, nginx luôn luôn map IP đó với một server cố định duy nhất (điều này được thực hiện với các stateful server nhằm mục đích lưu login state, session)

Ta thêm directive `ip_hash` như sau:

```nginx
upstream express_servers {
  ip_hash;
  server localhost:1111;
  server localhost:2222;
  server localhost:3333;
}
```

Thực hiện start up 3 servers tương tự như ở phần `load-balancer` trên, tiến hành chạy lệnh

```shell
while sleep 1; do curl http://localhost:8888; done
```

Ta thu được kết quả như sau:

![Screen Shot 2022-09-23 at 16 42 11](https://user-images.githubusercontent.com/15076665/191913281-8660f1d0-3784-4a07-8c4a-1da1609d13f0.png)

thấy rõ rằng mọi requests đều chỉ đi đến server 2

### Phân bổ request dựa theo active connection hoặc load

Thay vì chọn ra server kế tiếp thì nginx sẽ tìm đến server với số lượng `active connection` ít nhất (nói cách khác là server "rảnh nhất").

Ở ví dụ lần này thay vì sử dụng directive `ip_hash` ta sẽ sử dụng `least_conn;`. Chỉnh sửa một chút về môi trường, ở server 1 ta sẽ cho nó `sleep` chừng khoảng 20 giây trước khi trả về response.

```shell
while sleep 1; do curl http://localhost:8888; done
```

Chạy câu lệnh phía trên ở 2 terminals khác nhau chúng ta sẽ thấy rõ sự khác biệt, ở terminal đầu tiên chúng ta bị "mắc kẹt" do server 1 đang "sleep" chừng 20 giây

![Screen Shot 2022-09-23 at 16 55 37](https://user-images.githubusercontent.com/15076665/191915905-146e8497-a2ee-4d66-bb32-0d18475e674c.png)

Còn ở terminal 2 thay vì "đi đến" server 1, request đã đi đến 2 servers còn lại là 2 và 3 để tránh server 1 "đang hoạt động"

![Screen Shot 2022-09-23 at 16 55 23](https://user-images.githubusercontent.com/15076665/191915921-fe660121-814d-4658-be86-6dbaf2d02e4e.png)
