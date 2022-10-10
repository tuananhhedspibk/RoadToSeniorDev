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
