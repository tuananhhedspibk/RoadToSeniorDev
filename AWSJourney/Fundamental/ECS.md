# ECS Journey

Nội dung được tham khảo từ khoá học: <https://acloudguru.com/course/aws-certified-solutions-architect-associate-saa-c02>

## Definition

ECS là công cụ để quản lí các EC2 instances hoặc Fargate instances thông qua các `clusters` (Mỗi cluster sẽ có các instances bên trong)

![Screen Shot 2022-10-08 at 11 39 01](https://user-images.githubusercontent.com/15076665/194683998-ac9e2b42-6317-4570-822f-728ff031be35.png)

## Terminoloies

### Task definition, Container definition

Task definition sẽ định nghĩa app (tương tự như Dockerfile). Bên trong task sẽ có các containers. Các containers này được định nghĩa thông qua `container definition` - đây là các containers mà task sẽ sử dụng

> Mỗi một container chạy ta sẽ có 1 task tương ứng với nó

Có thể định nghĩa `min`, `max` task number bằng `Service`

### Registry

Storage cho container images (VD: ECR, docker hub, ...)

### Scope & relate

![Screen Shot 2022-10-08 at 12 12 11](https://user-images.githubusercontent.com/15076665/194685283-1630a3d2-1b8b-425c-a170-871b035e925c.png)

## Fargate

- `Serverless` container engine
- Pay for ressouce per app

## EC2 or Fargate

Chọn EC2 nếu:

- Tuân thủ requirement
- Yêu cầu broader customization
- Yêu cầu GPUs

## EKS (Elastic Kubernetes Service)

- Tương tự như ECS nhưng quản lí các containers trong `pods`
- Support EC2, Fargate
- Sử dụng K8s

## ECR

- Lưu trữ, deploy các images

![IMG_0857](https://user-images.githubusercontent.com/15076665/194686312-9282b6f1-3132-425a-971f-2c12e68642a1.jpg)

## ECS + ELB

- ELB (Elastic Load balancer) sẽ điều phối các requests đến server tương ứng phù hợp nhất
- ECS hỗ trợ ALB, NLB, CLB
- ALB sẽ điều phối HTTP/ HTTPS (layer 7) traffic
- NLB, CLB sẽ điều phối TCP (layer 4) traffic
- ALB cho phép:
  - Dynamic host port mapping
  - Path-based routing
  - Priority rules

![Screen Shot 2022-10-08 at 11 53 58](https://user-images.githubusercontent.com/15076665/194684706-d8879cb3-7fce-4cc6-a250-f4181c8e0787.png)

## ECS security

ECS cho phép gán role cho từng task bên trong 1 EC2 instance như hình minh hoạ bên dưới:

![Screen Shot 2022-10-08 at 12 03 50](https://user-images.githubusercontent.com/15076665/194685136-06f87313-551a-4373-abf7-66c8d17485c7.png)
