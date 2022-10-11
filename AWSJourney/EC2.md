# Quickly recap about EC2

## Security group

Security group là `stateful` nên do đó ta không cần quan tâm nhiều đến việc mapping giữa `inbound` và `outbound` (do mọi outbound tương ứng với inbound sẽ được tạo tự động).

Mọi `inbound traffic` trong Security group là `mặc định block`.

Mọi `outbond traffic` được cho phép.

Không thể setup `deny rules` với security group.

> Việc block IP, port là không thể đối với Security Group

## HA architeture

### Load balancer

- Application LB: HTTP, HTTPs, hoạt động ở layer 7 trong OSI model
- Network LB: TCP traffic, hoạt động ở layer 4 (connection level)
- Classic LB: có thể cân bằng tải của HTTP/HTTPS app ở layer 7 (X-Forwarđe, sticky session), ngoài ra cũng có thể sử dụng cho cả layer 4 (với TCP protocol)

### Error in Load balancing

Với `Classic LB`, khi app dừng responding, ELB sẽ trả về 504 error (nguyên nhân có thể do web server hoặc DB layer)

`X-Forwarded-For header` sẽ cho phép ta biết được IP từ phía client

### Target group

Sẽ chứa trong nó các `targets` (có thể là lambdas hoặc ec2 instances).

Target group là nơi mà load balancer sẽ route các requests đến các targets bên trong đó để tiến hành `Health check`

Target group sẽ nói cho load balancer biết "địa chỉ" để "chuyển" traffic tới (có thể là EC2 instances hoặc fixed IP address).

Lấy ví dụ về `blue/green deployment`, giả sử các instances mới sẽ thuộc về phía `blue target group` (ở đây ta có 2 target groups là `blue` và `green`), trong quá trình deploy `blue` thì traffic **SẼ ĐƯỢC CHUYỂN DẦN** từ `green` (hiện tại) sang `blue` (mới).

Việc "chuyển dần" này sẽ do `load balancer` thực hiện nhưng dựa theo các thông tin từ phía `target group`

![illustration-2](https://user-images.githubusercontent.com/15076665/195119477-6b38145e-bb40-463e-861f-589733f10064.png)

## Auto scaling

Auto scaling có 3 thành phần chính:

- Groups: Database group, Webserver group, ... Nói chung đây là nơi đặt các EC2 components
- Configuration templates: launch template như là `key pair`, `security group`, `AMI ID`, ...
- Scaling options: cung cấp các cách để scale auto scaling group. VD: scale dựa theo các điều kiện hoặc theo một schedule cho trước

### Các scaling options

![Screen Shot 2022-10-11 at 23 10 35](https://user-images.githubusercontent.com/15076665/195114715-75da2e0d-9dd8-4120-81c6-fd8fd7a86450.png)

#### Maintain current instance levels at all times

Configure auto scaling luôn luôn đảm bảo một số lượng nhất định các instances luôn luôn chạy ở mọi thời điểm.

Để thực hiện điều này, `Auto scaling` sẽ tiến hành `health check` theo từng khoảng thời gian nhất định, nếu phát hiện bất kì `unhealthy` instance nào thì instance đó sẽ bị thay thể bởi một `healthy` instance

#### Scale manually

Ở đây ta chỉ thay đổi số lượng `max`, `min`, `desired capacity` các instances.

#### Scale base on schedule

Scaling tự động dựa theo datetime

#### Scale based on demand

Định nghĩa các params điểu khiển quá trình scale

#### Use predictive scaling

Sử dụng kết hợp `EC2 auto scaling` + `AWS auto scaling`

