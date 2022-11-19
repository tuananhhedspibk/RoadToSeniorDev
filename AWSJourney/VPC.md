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

## Bastion host

`NAT gateway` không cần phải đứng phía sau Security group như `NAT instance`.

> Cả NAT gateway và NAT instance có chức năng cung cấp internet traffic cho EC2 instance trong private subnet

Bastion như một cách để ta có thể truy cập vào EC2 instance trong private subnet thông qua SSH hoặc RDP

Bastion sẽ được "làm cứng" để đảm bảo an toàn tối đa cho private instance phía sau nó

> Không thể sử dụng NAT gateway làm Bastion host

Minh hoạ về Bastion host:

![Screen Shot 2022-11-03 at 22 34 26](https://user-images.githubusercontent.com/15076665/199734885-3f38b544-7e85-40ac-9929-43e9de054ac5.png)

## Direct connect

Tạo kết nối trực tiếp từ premise của người dùng tới AWS mà không thông qua Internet-based connection

High throughput workloads

![Screen Shot 2022-11-03 at 22 41 13](https://user-images.githubusercontent.com/15076665/199736356-2801100c-81b0-4da2-afff-6a2ce275f0a5.png)

## VPC Endpoint

Cho phép VPC của ta có thể kết nối (một cách private) đến AWS services mà không cần thông qua internet.

2 loại:

- Interface Endpoint: Elastic network interface với private IP, hoạt động như entry point vào các AWS services mà VPC muốn kết nối tới
- Gateway Endpoint: hỗ trợ S3, DynamoDB

Trước khi sử dụng Gateway endpoint, Private instance muốn kết nối tới S3 bucket thì phải thông qua `NAT gateway` để đi ra `internet` rồi mới đến được S3 bucket như hình bên dưới.

![Screen Shot 2022-11-19 at 13 04 41](https://user-images.githubusercontent.com/15076665/202833389-47252a62-491e-4c71-9149-1000ce112a6c.png)

Với `Gateway Endpoint` thì không cần phải đi ra ngoài internet mà vẫn có thể kết nối tới S3 bucket được.

![Screen Shot 2022-11-19 at 13 09 23](https://user-images.githubusercontent.com/15076665/202833510-3e618941-d373-4917-8515-52dab27fd482.png)

Endpoint về bản chất là virtual devices. Chúng horizontally scaled

## VPC Peering

VPC peering connection là kết nối mạng giữa 2 VPCs trực tiếp (sử dụng private IP address) dựa trên infrastructure của VPC chứ KHÔNG HỀ thông qua:

- Gateway
- VPN connection

Các instances trong các VPCs có thể tương tác trực tiếp với nhau giống như thể chúng nằm trong cùng 1 network.

![Screen Shot 2022-11-19 at 16 06 36](https://user-images.githubusercontent.com/15076665/202839281-74e552b4-a988-4444-b0b6-7e82b0608495.png)

Đồng thời VPC peering connection cũng không dựa vào bất kì thiết bị vật lý nào nên tránh được:

- Bottleneck
- SPOF

Thông qua VPC peering connection, các VPCs có thể kết nối với nhau dù ở khác region. Lúc này các kết nối sẽ dựa trên AWS backbone chứ không hề thông qua internet, giúp tránh DDoS attack, trễ, ...

## Private link

Để share app trên nhiều VPC ta có thể:

- Open VPC với internet, tuy nhiên cách làm này sẽ khiến cho mọi resource trong public subnet sẽ được public ra bên ngoài.
- Sử dụng `VPC peering`: quản lí các peering relationships giữa các VPCs

Với private link:

- Ta không cần VPC peering, không cần route table, NAT, IGWs
- Tuy nhiên ta cần NLB trên service VPC và Elastic Network Interface trên customer VPC

![Screen Shot 2022-11-19 at 15 51 47](https://user-images.githubusercontent.com/15076665/202838645-ce068e9a-5ea1-43bd-b8d1-de9f4f725893.png)

> Với trường hợp cần VPC peering cho 10, 100, 1000 customer VPC hãy nghĩ đến AWS PrivateLink

## Transit Gateway

Hoạt động như một Hub trung chuyển giữa các VPCs, Direct Connect Gateway, VPN connection, ...

![Screen Shot 2022-11-19 at 16 10 00](https://user-images.githubusercontent.com/15076665/202839406-effeae88-6c81-41d6-9193-61ccf5422705.png)

Có thể sử dụng Route table để hạn chế giao tiếp giữa các VPCs.

## VPN Cloudhub

Sử dụng để kết nối các sites (mỗi site sẽ có một VPN connection riêng).

Hub-and-spoke model.

Các kết nối giữa customer gateway và VPN Cloudhub luôn được mã hoá.

## Network cost

Sử dụng Private IP sẽ tiết kiệm chi phí hơn so với Public IP.

![Screen Shot 2022-11-19 at 16 16 37](https://user-images.githubusercontent.com/15076665/202839628-cbfc2d45-82ef-4c4d-8c18-8fb35c8d3c7f.png)

Việc dồn mọi instance vào cùng một zone và sử dụng private IP address có thể sẽ giúp tiết kiệm chi phí (gần như free về traffic) nhưng sẽ dẫn đến nguy cơ SPOF.
