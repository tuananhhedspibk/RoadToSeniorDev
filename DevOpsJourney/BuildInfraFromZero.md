# Tôi đã xây dựng infrastructure (AWS) cho hệ thống micro-service từ zero bằng terraform như thế nào ?

## Giới thiệu qua về terraform

Terraform là một công cụ **Infrastructure As Code - (IaC)** cho phép bạn có thể tạo và quản lí các infrastructure resource bằng code.

Trong terraform có một vài khái niệm cơ bản sau cần phải nắm vững:

- Terraform state
- Terraform resource
- Terraform variable
- Terraform data source

### Terraform state

Có thể hiểu đây như là một công cụ để terraform theo dõi metadata cũng như tham chiếu giữa infrastructure resource trong thực tế và configure hiện thời của bạn.

Minh hoạ như hình dưới đây

![Screen Shot 2023-12-07 at 14 55 54](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/371205e9-5771-49df-b9d8-d8c88ff6bd2f)

Không những thế Terraform state còn phát huy sức mạnh khi tiến hành làm việc nhóm, nói một cách đơn giản dựa theo terraform state, các thành viên trong team có thể biết được:

- Configure của mình "khác" gì so với resources trong thực tế.
- Configure của mình "xung đột" như thế nào so với resources trong thực tế.

Terraform state được lưu trong file `terraform.tfstate` dưới JSON format. Tuy nhiên ta nên lưu trữ nó trên môi trường cloud và mã hoá cẩn thận.

### Terraform resource

Resource là element cơ bản trong Terraform language, mỗi resource block mô tả một hoặc nhiều infrastructure objects như Virtual-network, Compute-instances.

```terraform
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr

  enable_dns_hostnames = true
  enable_dns_support   = true

  tags   = {
    Name = "${var.app_name}-vpc"
  }
}
```

Như ví dụ ở trên đây tôi có một `aws_vpc` với tên là `main`, trong đó các thông tin về

- cidr_block
- tags
- dns_hostnames
- ...

đều được chỉ ra một cách cụ thể.

### Terraform variable

Tương tự như các ngôn ngữ lập trình (Programming Language) khác, terraform language cũng sở hữu cho mình các định nghĩa riêng về biến.

Trong terraform có 2 loại biến cơ bản:

- Biến input - Input Variable
- Biến local - Local Variable

```terraform
variable app_name {
  type = string
}
```

**Biến input** sẽ được khai báo theo format như trên (sử dụng từ khoá `variable` và chỉ ra `type` của biến), các biến này sẽ định nghĩa các "tham số đầu vào" của một module. Ví dụ:

```terraform
// Module A

variable app_name {
  type = string
}
```

Ở đoạn code trên tôi định nghĩa một Module A, module cần "tham số đầu vào" là **app_name**. Các module khác muốn sử dụng module A này thực hiện đoạn config như sau:

```terraform
module "A" {
  source   = "./module_A"

  app_name = "Sample App"
}
```

**Biến local** sẽ được khai báo theo như format dưới đây

```terraform
locals {
  test_variable = "test"
}
```

Các biến local này sẽ chỉ có tác dụng trong nội bộ module mà nó được định nghĩa mà thôi.

### Terraform data source

Data source cho phép chúng ta có thể sử dụng các resources **KHÔNG ĐƯỢC ĐINH NGHĨA BỞI terraform**, ví dụ như tham chiếu đến các infrastructure resources có sẵn trong hệ thống.

Theo kinh nghiệm cá nhân, tôi thấy data source khá hữu dụng đặc biệt với các hệ thống mà trước đây việc setup infrastructure thường được triển khai bằng "tay" thay vì tổ chức bằng code. Data source cho phép chúng ta có thể tham chiếu đến các "legacy resources" này.

```terraform
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```

## Kiến trúc infrastructure cho hệ thống micro-service

Trong lần này tôi tiến hành triển khai 3 micro-services: A, B, C.

Với từng micro-service tôi sẽ tiến hành thực thi theo kiến trúc cơ bản như sau:

**Kiến trúc mạng** (vpc, public_subnet, private_subnet)

![Screen Shot 2023-12-07 at 22 22 27](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/da08fd8d-4644-4a4b-ab7b-112b1fe44791)

Phân tích từng thành phần trong hình vẽ trên. Ở đây với mỗi service tôi sẽ tạo **riêng một vpc**.

Trong mỗi vpc, tất nhiên rồi sẽ không thể thiếu đi **public_subnet** và **private_subnet**. Và đương nhiên, để các resources trong **private_subnet** có thể đi ra mạng internet bên ngoài tôi sẽ thiết lập một nat_gateway và đặt nó bên trong **public_subnet**.

Database (ở đây là AWS RDS) sẽ được đặt bên trong **private_subnet** và chỉ chấp nhận 2 đầu vào:

- Hoặc là trong nội bộ vpc mà thôi.
- Hoặc là Proxy nếu đi từ internet vào.

**AWS ECS** để chạy server code của service sẽ được đặt bên trong **private_subnet**.

**Kiến trúc ứng dụng** (ECS, Load Balancer)

![Screen Shot 2023-12-07 at 22 30 17](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/7334d9cf-345d-4f41-9f90-d0496814e654)

Ở đây tôi chỉ thuần tuý đặt phía trước các ECS một **Application Load Balancer**, bản thân bên trong ECS tôi cũng đặt một **nginx load balancer** đóng vai trò **reverse proxy** bảo vệ cho server code chạy bên trong ECS service.

## Tham chiếu code

Giờ có lẽ là phần được mong đợi nhất, đó chính là triển khai terraform coding.

Các bạn có thể tham khảo full source code sample tại: <https://github.com/tuananhhedspibk/NewAnigram-Infrastructure>

### Tổ chức code

Tôi chia các resources theo đơn vị module:

- network: vpc, public_subnet, private_subnet
- ecs_api: ecs_task_definition, ecs_service
- ecs_cluster: cluster
- proxy: database proxy
- rds: database

Bao ngoài cùng sẽ là một file `main.tf` như sau:

```terraform
module "network" {
  source = "./network"
  // ...
}

module "proxy" {
  source = "./proxy"
  // ...
}

module "rds" {
  source = "./rds"
  // ...
}

module "ecs_cluster" {
  source = "./ecs_cluster"
  // ...
}

module "ecs_api" {
  source = "./ecs_api"
  // ...
}
```

### Network module

Trong module này tôi sẽ định nghĩa:

- vpc
- public_subnet
- private_subnet
- nat_gateway

Với `vpc` sẽ là:

```terraform
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "${var.app_name}-vpc"
  }
}
```

việc quan trọng nhất khi định nghĩa vpc đó là chỉ ra `cidr_block` cho nó. `cidr_block` có thể hiểu như địa chỉ mạng (địa chỉ ảo) của vpc.

Với `public_subnet` sẽ như sau:

```terraform
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id // id của vpc resource

  count                   = length(var.public_subnets_cidr) // số lượng public_subnet
  cidr_block              = element(var.public_subnets_cidr, count.index) // thiết lập cidr_block - địa chỉ mạng con cho các subnet
  availability_zone       = element(var.availability_zones, count.index)
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.app_name}-public-subnet-${element(var.availability_zones, count.index)}"
  }
}
```

do là **public_subnet** - tức là **CÓ THỂ** nhìn thấy mạng con này từ global internet, nên ta cần thiết lập thuộc tính `map_public_ip_on_launch` với giá trị `true` để mạng con này có public IP từ đó global internet có thể nhìn thấy nó.

Tương tự với `private_subnet`:

```terraform
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id // id của vpc resource

  count                   = length(var.private_subnets_cidr) // số lượng private_subnet
  cidr_block              = element(var.private_subnets_cidr, count.index) // thiết lập cidr_block - địa chỉ mạng con cho các subnet
  availability_zone       = element(var.availability_zones, count.index)
  map_public_ip_on_launch = false

  tags = {
    Name = "${var.app_name}-public-subnet-${element(var.availability_zones, count.index)}"
  }
}
```

do là **private_subnet** - tức là **KHÔNG THỂ** nhìn thấy mạng con này từ global internet, nên ta cần thiết lập thuộc tính `map_public_ip_on_launch` với giá trị `false` để mạng con này **KHÔNG CÓ public IP** từ đó global internet **KHÔNG THỂ** nhìn thấy nó.

Còn với **nat_gateway** tôi sẽ làm như sau:

## Tổng kết
