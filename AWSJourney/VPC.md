# Quickly recap about VPC (Virtual private cloud)

## What is VPC

Có thể coi như một `data centre` trên cloud

![Screen Shot 2022-10-10 at 11 53 54](https://user-images.githubusercontent.com/15076665/194793366-0e1bbb12-df91-4773-8188-a493209f5de9.png)

1 subtnet = 1 Availability zone

> Security group: stateful, ACL: stateless

Route table cho phép các subnets trong VPC có thể tương tác qua lại thông qua route address

![Screen Shot 2022-10-10 at 12 28 28](https://user-images.githubusercontent.com/15076665/194795520-b9925eb9-55a9-4856-a434-8d9e53093fcf.png)

Khi VPC được tạo ra, nó luôn liên kết với một `main route table`

## Star Peering Connection

Các VPCs có thể kết nối theo kiến trúc hình sao

![Screen Shot 2022-10-10 at 12 01 59](https://user-images.githubusercontent.com/15076665/194793857-b6544d06-eee0-49df-b828-ba9c37fcb405.png)

Tuy nhiên kết nối này lại không có tính bắc cầu (như ở hình trên VPC B -> VPC A -> VPC C, không có nghĩa là VPC B có thể kết nối tới VPC C)

## Available IP addresses

Trong 1 VPC luôn bị "khuyết đi" 5 addresses:

- Network address
- Broadcast address
- VPC router address (use by AWS)
- DNS server address (used by AWS)
- Future use address (use by AWS)

## NAT instance, NAT gateway

Mục đích chính đều là để các private subnet có thể đi ra ngoài internet mà không cần public ra ngoài

### NAT instance

![Screen Shot 2022-10-10 at 13 13 08](https://user-images.githubusercontent.com/15076665/194798413-f6b77b92-e611-4973-bd13-879ec1042a9d.png)

Ta sẽ sử dụng EC2 instance làm NAT.

Mặc định thì `EC2 instance` sẽ **CHO PHÉP check source/ destination** (vì khi đó EC2 instance hoặc sẽ là source hoặc sẽ là destination).

Nhưng với `NAT instance` thì khác, nó sẽ đóng vai trò nhận và gửi request/ traffic **KHÔNG PHẢI CỦA NÓ** nên với `NAT instance` ta sẽ **DISABLE heck source/ destination**

NAT instance sẽ hoạt động như một chiếc cầu nối giữa `private subnet` và `internet gateway` để giúp `private subnet` có thể đi ra ngoài internet

NAT instance **PHẢI ĐƯỢC ĐẶT TRONG public subnet**

Trong trường hợp `bottle neck`, ta cần tăng `instance size`, lúc nào `NAT instance` cũng cần đứng sau một `Security group`

### NAT gateway

Highly available gateway

![Screen Shot 2022-10-10 at 13 13 08](https://user-images.githubusercontent.com/15076665/194798413-f6b77b92-e611-4973-bd13-879ec1042a9d.png)

Chỉ tồn tại trong 1 AZ duy nhất và **không kết nối** đến `security group`

Trong trường hợp nhiều resource cùng sử dụng chung một `NAT gateway` thì sẽ dễ xảy ra `SPOF` - tức là nếu AZ chứa NAT gateway bị sập thì các AZs còn lại cũng sẽ không thể kết nối tới internet được như minh hoạ ở hình dưới

![Screen Shot 2022-10-10 at 16 06 16](https://user-images.githubusercontent.com/15076665/194814124-22f552a6-9adc-4747-8755-53436ef898bd.png)

Giải pháp ở đây đó là phát triển các **NAT gateway độc lập riêng** cho từng AZ như hình phía dưới

![Screen Shot 2022-10-10 at 16 06 11](https://user-images.githubusercontent.com/15076665/194814122-2c64bca3-91e5-4cf5-868e-e701330765cf.png)

## Network ACL

Mặc định khi mới tạo, network ACL inbound và outbound sẽ deny mọi access đến và đi

`1024-65535`: ephemeral port - là một dải các ports dùng cho các communication session trong thời gian ngắn, khi không được sử dụng nữa port sẽ được giải phóng để dùng cho các session khác.
NAT Gateway sử dụng dải port như trên.

Trong network ACL các rules ở phía trên sẽ có priority cao hơn.

![Screen Shot 2022-10-10 at 16 41 32](https://user-images.githubusercontent.com/15076665/194818929-8e226708-2a9f-4f8b-996d-772a8d00d8d3.png)

Nếu muốn `deny` thì phải đặt `deny` ở **PHÍA TRƯỚC** `allow`

1 subnet chỉ kết nối với 1 network ACL duy nhất.

Network ACL luôn đứng phía trước security group.

Trong trường hợp muốn block IP, ta cần tiến hành ở `Network ACL`

Network ACL là `stateless` nên các outbound traffic cần map với inbound traffic
