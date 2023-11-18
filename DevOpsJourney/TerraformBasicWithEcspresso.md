# Terraform basic concepts with ecspresso

## Setup ứng dụng bằng AWS

Về cơ bản ta sẽ sử dụng ECS như "xương sống" để thiết lập ứng dụng.

ECS là một orchestration service cho phép:

- Deploy
- Manage
- Scale

các containerized app.

Trong ECS có 2 loại capacity chính:

- EC2 instance: quản lí các servers dưới dạng EC2.
- Fargate: Serverless - dùng bao nhiêu trả bấy nhiêu và không cần phải tự quản lí các servers như EC2 instance phía trên.

Vòng đời của một ECS application

Trong đó:

![Screen Shot 2023-11-18 at 22 42 22](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/70305f62-d7d1-4eb2-9ee9-cddcdab3b854)

- ECR sẽ lưu DockerImage tương ứng với app.
- Task Definition sẽ là blueprint của app (JSON file với các params, containers cấu thành nên app)

Ngoài ECS còn các service ở tầng:

- Application: EC2, Lambda, Load balancer
- Database: RDS, ElastiCache, ...
- Network: VPC

## Terraform state

Là một JSON file được sinh ra sau khi chạy lệnh `terraform apply`.

Nó sẽ lưu giữ các mapping giữa `logic resource trong terraform configure file` và `resource được tạo ra trên cloud provider`.

File state này sẽ giúp:

- Quản lí resource state.
- Tránh conflict khi có nhiều người cùng quản lí và dev infra resource bằng code.

## ecspresso

Là một công cụ để deploy ECS.

Về cơ bản cần những config:

- Task Definition
- Service Definition

ecpresso chỉ quản lí những tài nguyên như hình bên dưới

![ai0zqvcw28txeu6ve3iqxfrds0as](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/b6403cf4-1137-44ad-89c5-e9081bf4c1d9)

Chú ý rằng, ta có thể deploy ứng dụng ECS bằng ecspresso, mỗi lần deploy là mỗi lần một task definition mới được sinh ra từ ECR tương ứng.
