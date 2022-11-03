# ELB

## Fundamental

Bản thân các load balancer cũng thực hiện `health check` với các instance mà nó forward downstream tới.

![Screen Shot 2022-10-31 at 22 35 20](https://user-images.githubusercontent.com/15076665/199020527-0ee380f6-0aa5-48d9-8cb0-ee14574f1f03.png)

health check thường được thực hiện trên:

- Port
- Route (thường là `/health`)

Instance response `200` sẽ được coi là `healthy`, các trường hợp khác là `unhealthy`

User có thể truy cập tới load balancer `từ bất cứ đâu` nên `security group` của load balancer sẽ là:

- HTTP: 0.0.0.0/0
- HTTPS: 0.0.0.0/0

Với instance thì `security group` của nó sẽ có `inbound` là `Load balancer`

![Screen Shot 2022-10-31 at 22 41 21](https://user-images.githubusercontent.com/15076665/199021778-124f7a94-d157-4251-9515-32b1d4e474af.png)

## Application Load Balancer

- Nằm ở tầng 7 (OSI), áp dụng cho HTTP
- Load balancing tới nhiều HTTP applications qua target group
- Support HTTP/2, WS
- Support redirect

Có thể routing đến các target groups khác nhau tuỳ theo:

- path trong URL (example.com`/user` & example.com`/post`)
- hostname (`example.com`, `other.com`)
- query string, header (example.com/users?`id=2`)

Phù hợp với `microservice` hoặc `container-based application - (Docker, ECS)`

Routing base URL path:

![Screen Shot 2022-10-31 at 23 15 48](https://user-images.githubusercontent.com/15076665/199029193-f4643653-66a5-4b7e-9683-6a6b5878691d.png)

Target group của ALB:

- EC2 instances
- ECS tasks
- Lambda function, HTTP req sẽ được convert thành JSON object event
- IP addresses (Private IPs)

Health check sẽ được thực hiện ở `target group level`

Application server sẽ không nhìn thấy trực tiếp `IP address` của client mà thay vào đó sẽ phải xem qua `X-Forwarded-For`

Chúng ta cũng có thể lấy về port và protocol thông qua lần lượt `X-Forwarded-Port` & `X-Forwarded-Proto`

## Network Load Balancer

Nằm ở layer 4. Cho phép:

- Forward TCP, UDP traffic tới instances
- Xử lí cả triệu req trên giây
- Less latency ~ 100ms

NLB có một static IP trên mỗi AZ, hỗ trợ assign Elastic IP

Target groups:

- EC2 instances
- IP addresses (Private IPs)
- ALB
- Health check support TCP, HTTP, HTTPS

![Screen Shot 2022-11-01 at 8 27 05](https://user-images.githubusercontent.com/15076665/199128314-1b8d4e68-264e-4f46-b9e2-325fcea706a5.png)

## SSL/TLS

Cho phép mã hoá traffic giữa client & LB (in-flight encryption)

SSL = Secure Socket Layer
TLS = Transport Layer security (new version of SSL)

Các certificate này sẽ được quản lí thông qua ACM

![Screen Shot 2022-11-03 at 11 30 52](https://user-images.githubusercontent.com/15076665/199638365-e3e0ed1e-1295-41d9-bed2-b47cf8b420df.png)

### SNI - Server name indication

SNI giải quyết vấn đề sử dụng nhiều SSL certificates trên một web server

Đây là một `protocol mới`, yêu cầu client phải chỉ rõ hostname trong quá trình SSL handshake

Server sau đó sẽ tìm certificate phù hợp hoặc trả về default certificate

![Screen Shot 2022-11-03 at 11 43 58](https://user-images.githubusercontent.com/15076665/199638353-5b6ec3ac-981c-4ce2-8ded-7279f2cc7741.png)

Tuy nhiên SNI chỉ hoạt động với ALB, NLB (newer generation), không hoạt động với CLB (old generation)

### ELB - SSL Certificates

- CLB chỉ hỗ trợ SSL certificate, tuy nhiên phải sử dụng nhiều CLBs cho nhiều hosts với nhiều certificates
- ALB hỗ trợ multiple listeners với multiple SSL certificates, quản lí các certificates bằng SNI
- NLB hỗ trợ multiple listeners với multiple SSL certificates, quản lí các certificates bằng SNI

### Connection draining

Feature naming:

- Connection draining với CLB
- Deregistration Delay với ALB, NLB

Về bản chất nó là khoảng thời gian để một `de-registering` hoặc `unhealthy` instance có thể hoàn thiện các `in-flight requests`.

Sau đó sẽ ngưng việc gửi request tới các `unhealthy` hoặc `de-registering` instances.

![Screen Shot 2022-11-03 at 12 03 26](https://user-images.githubusercontent.com/15076665/199640177-ab111d55-ee75-4006-bd18-bc08f58fedb9.png)

Khoảng thời gian này sẽ từ 1 ~ 3600s
