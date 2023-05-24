# Private Integration with Api Gateway And VPC Link

Ta thường sử dụng sự kết hợp này khi muốn xây dựng Api Gateway kết nối với các private resource như EC2 hay ECS.

Ta thường sẽ kết hợp sử dụng với NLB. Việc thiết lập private inegration kiểu này sẽ giúp các HTTPs resources bên trong VPC có thể được truy cập từ bên ngoài VPC.ư

## HTTP API & ALB

![Screenshot 2023-05-23 at 14 52 52](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/79828986/b6be21a5-4121-4f1e-9675-6d3a1d9530d4)

