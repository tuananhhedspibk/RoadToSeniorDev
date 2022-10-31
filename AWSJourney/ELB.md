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
