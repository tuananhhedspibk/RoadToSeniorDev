# Quickly recap about EC2

## EC2 instances type

- `On-demand`: default mode, khi request instance nếu còn capacity thì sẽ cấp phát instance, instance là của chúng ta một khi được tạo
- `Reserved`: AWS sẽ phục vụ cho chúng ta
- `Spot`: chúng ta yêu cầu instance, cung cấp maximum price và nếu còn free capacity và price của chúng ta >= price hiện tại thì ta sẽ có instance

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

### Một vài scaling policies

#### Target tracking policy

Scale out Auto scaling group khi một metric cụ thể nào đó có sự thay đổi

#### Simple scaling & Step scaling

Chọn scaling metrics và threshold value cho Cloudwatch alarm để có thể tiến hành quá trình scaling

#### Suspend & Resume scaling

Dừng việc scaling tạm thời có thể do:

- Thay đổi file configure
- ...

Và đồng thời có thể tiếp tục quá trình scaling lại.

## Cloudwatch

Host level metrics:

- CPU
- Network
- Disk
- Status check

Standard monitoring: 5 phút

Detail monitoring: 1 phút

## Cloudtrail

Tăng cườngg sự giám sát với user và resource activity thông qua việc recording:

- AWS Management Console actions
- API calls

Từ đó ta có thể biết được user nào, account nào, IP nào thực hiện AWS calling hay API calling

> Cloudwatch monitoring performance, Cloudtrail monitor API calls in AWS platform

## Storage

### EBS (Elastic Block Store)

Network drive gắn với instance, cho phép lưu trữ dữ liệu lâu dài kể cả khi instance bị tắt (terminated).

Chỉ có thể mount với 1 instance duy nhất (CCP level).

EBS chỉ thuộc về 1 AZ duy nhất.

EBS cũng có thể được attached vào hoặc dettached khỏi 1 instance

> Pricing for provision capacity (GBs, IOPS)

Mặc định thì `root EBS volume` sẽ bị xoá khi EC2 instance tắt (Delete on termination) các `EBS volume` khác sẽ không bị xoá khi EC2 instance tắt.

### EBS snapshot

Là bản backup của EBS tại một thời điểm nhất định. Snapshot có thể được copy trên nhiều AZs hoặc regions.

> Không nhất thiết phải dettach volume khi lấy snapshot nhưng nên làm như vậy

![Screen Shot 2022-10-31 at 8 26 29](https://user-images.githubusercontent.com/15076665/198907077-e275a903-7f3c-4822-a30b-d83f20729f5c.png)

Một vài features:

- Snapshot Archive:
  - Chuyển sang `archive tier` sẽ rẻ hơn `75%`
  - Cần 24 - 72 giờ để restoring archive
- Recycling: ta có thể setup rules để recover lại snapshot khi bị `accident deleted`, có thể chỉ định `retention` từ 1 ngày - 1 năm
- Fasst snapshot restore (FSR): force full initialization để không còn latency khi sử dụng lại lần đầu (khá tốn $)

### EBS volume types

Có 6 types:

- gp2/gp3 (SSD): SSD volume phổ biến
- io1/io2 (SSD): Highest-performance SSD (low-latency, high throughput)
- st1 (HDD): Low cost HDD volume (frequently accessed)
- sc1 (HDD): Lowest cost HDD volume (infrequently accessed)

EBS volumes được đặc tả thông qua `Size` | `Throughtput` | `IOPS (I/O Per Second)`

> Chỉ có gp2/gp3 và io1/io2 có thể sử dụng làm boot volume cho EC2 instance

### EBS Multi-Attach - io1/io2 family

Trong cùng một AZ, ta có thể attach 1 EBS volume cho nhiều EC2 instances

![Screen Shot 2022-11-03 at 12 41 35](https://user-images.githubusercontent.com/15076665/199643616-9b88c8ae-217e-4587-8fc4-9d2cd297b979.png)

Mỗi instance có full read/ write permission với EBS volume

## EFS - Elastic File System

Quản lí NFS (network file system) có thể được mount đến các instances thuộc nhiều AZs

![Screen Shot 2022-11-03 at 12 49 38](https://user-images.githubusercontent.com/15076665/199644269-6c2d1db0-5bc9-4e30-a40d-f894ebfddfaf.png)

Sử dụng `Security group` để quản lí access đến EFS.

Usecases:

- Content management
- Web serving
- Data sharing
- Wordpress

### Storage class

**Storage Tiers:** dùng cho `frequenty access` files, `EFS-IA` (infrequently access): với các file không được truy cập trong 60 ngày (setup tuỳ theo `Lifecycle policy`) sẽ được di chuyển từ `EFS standard` sang `EFS-IA`

![Screen Shot 2022-11-03 at 13 04 31](https://user-images.githubusercontent.com/15076665/199645557-a3b7be50-2a38-414e-96e7-1592929e9311.png)

## EBS vs EFS

**EBS** chỉ có thể attach đến một EC2 instance duy nhất trong 1 AZ mà thôi. EBS cũng sẽ `được locked` vào một AZ duy nhất.

Ngoài ra ta có thể tạo snapshot cho EBS, sau đó sẽ tiến hành restore nó ở một AZ khác (migrate EBS)

![Screen Shot 2022-11-03 at 13 14 34](https://user-images.githubusercontent.com/15076665/199646466-b0faaa43-5736-4559-a957-80614ca24ec8.png)

**EFS** cho phép chia sẻ trên nhiều AZs, chỉ dùng cho Linux instance (POSIX)

![Screen Shot 2022-11-03 at 13 26 40](https://user-images.githubusercontent.com/15076665/199647605-23d2541c-aa88-4924-9617-493af7fef688.png)
