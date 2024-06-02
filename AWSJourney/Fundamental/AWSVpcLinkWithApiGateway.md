# Private Integration with Api Gateway And VPC Link

Ta thường sử dụng sự kết hợp này khi muốn xây dựng Api Gateway kết nối với các private resource như EC2 hay ECS.

Ta thường sẽ kết hợp sử dụng với NLB. Việc thiết lập private inegration kiểu này sẽ giúp các HTTPs resources bên trong VPC có thể được truy cập từ bên ngoài VPC.

## HTTP API & ALB

VPC Link giúp tiến hành thực thi private integration giữa HTTP APIs với private VPC resources.

![1-ALB-Example](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/79828986/d124b017-da4d-4b92-947a-a33dd86531db)

Với private integration, internal LB sẽ route các request dựa theo private IP bên trong private subnet
