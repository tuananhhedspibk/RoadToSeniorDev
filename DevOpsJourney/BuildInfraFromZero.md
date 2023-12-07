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

Mỗi một resource đều có các output (có thể hiểu như các public property của riêng mình).

Ví dụ:

Với AWS EC2 instance resource, sau khi được tạo ra nó sẽ có cho mình các thuộc tính cơ bản như:

- arn
- public_ip
- ...

Bạn đọc có thể tham khảo chi tiết hơn tại <https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance#attribute-reference>

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
_Hình 1_

Phân tích từng thành phần trong hình vẽ trên. Ở đây với mỗi service tôi sẽ tạo **riêng một vpc**.

Trong mỗi vpc, tất nhiên rồi sẽ không thể thiếu đi **public_subnet** và **private_subnet**. Và đương nhiên, để các resources trong **private_subnet** có thể đi ra mạng internet bên ngoài tôi sẽ thiết lập một nat_gateway và đặt nó bên trong **public_subnet**.

Database (ở đây là AWS RDS) sẽ được đặt bên trong **private_subnet** và chỉ chấp nhận 2 đầu vào:

- Hoặc là trong nội bộ vpc mà thôi.
- Hoặc là Proxy nếu đi từ internet vào.

**AWS ECS** để chạy server code của service sẽ được đặt bên trong **private_subnet**.

**Kiến trúc ứng dụng** (ECS, Load Balancer)

![Screen Shot 2023-12-07 at 22 30 17](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/7334d9cf-345d-4f41-9f90-d0496814e654)
_Hình 2_

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

```terraform
// ElasticIP cho _NAT _gateway
resource "aws_eip" "nat" {
  vpc        = true
  depends_on = [aws_internet_gateway.main]
}

// _NAT _gateway
resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = element(aws_subnet.public.*.id, 0) // config cho việc đặt nat_gateway tại public_subnet

  tags = {
    Name = "${var.app_name}-nat"
  }
}
```

Giải thích qua cho bạn đọc thì nat_gateway là một service cho phép các instances trong private_subnet có thể kết nối ra ngoài global internet và **KHÔNG CHO PHÉP** global internet truy cập vào instances bên trong private_subnet.

Cụ thể hơn bạn đọc có thể xem tại: <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html>

Trở lại với đoạn code thiết lập nat_gateway ở trên, tôi thiết lập:

- ElasticIP cho nat_gateway.
- Đặt nat_gateway tại public_subnet để từ đó instances trong private_subnet có thể đi ra global internet.

### Proxy module

Proxy ở đây thực chất chỉ là 1 EC2 instance mà thôi. Mục đích chính của tôi khi thiết lập một proxy "chắn" trước RDS đó là tạo một "cổng vào" cũng như "giới hạn" các truy cập (không tính truy cập từ phía server code) vào RDS.

Cụ thể như sau: ở **hình 1** phía trên các bạn có thể thấy RDS được đặt bên trong private_subnet nên **chắc chắn** từ global internet ta **KHÔNG THỂ** truy cập vào RDS được, do đó proxy ở đây sẽ hoạt động như một "cổng vào" khi dev muốn truy cập vào RDS để query cũng như chỉnh sửa dữ liệu.

![Screen Shot 2023-12-08 at 7 43 36](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/c3e89d04-0372-4dd0-bf67-5d66417c30f6)
_Hình 3_

Ở **hình 3**, bạn có thể thấy rằng dev sẽ truy cập vào RDS thông qua SSH Tunel, việc làm này:

- Vừa đảm bảo rằng dev có thể truy cập RDS.
- Vừa đảm bảo rằng số lượng người có thể truy cập vào RDS là giới hạn.

Do đó proxy sẽ hoạt động như một "tấm khiên" phía trước database của chúng ta.

Tôi sẽ thiết lập proxy như sau:

```terraform
resource "aws_security_group" "proxy" {
  name          = "${var.app_name}-proxy-sg"
  description   = "Allow ssh connect to db proxy"
  vpc_id        = var.vpc_id

  // Inbound rule
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  // Outbound rule
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.app_name}-proxy-sg"
  }
}

resource "aws_instance" "proxy" {
  instance_type   = "t2.micro"
  ami             = local.ami

  subnet_id       = var.subnet_id // public_subnet id
  security_groups = [aws_security_group.proxy.id]
  key_name        = "key_pair_name"

  tags = {
    Name = "${var.app_name}-db-proxy"
  }
}
```

Như đoạn code trên, để giới hạn chỉ cho phép truy cập vào proxy bằng SSH tôi thiết lập **inbound** cho **security_group** của proxy chỉ là cổng 22 mà thôi.

Do proxy chỉ đóng vai trò như một "vùng đệm" nằm phía trước RDS nên chỉ cần một **micro** instance là đủ. Và tất nhiên rồi, ta **BẮT BUỘC** phải đặt proxy trong public_subnet để có thể đi vào nó từ global internet.

### RDS module

Ở module này tôi tiến hành định nghĩa một RDS instance và cluster chứa instance đó. Mục đích chính là để lưu dữ liệu (nói cách khác đây chính là database của service).

```terraform
resource "aws_rds_cluster" "this" {
  cluster_identifier              = "${var.app_name}-mysql-cluster"
  engine                          = local.rds_engine
  engine_version                  = "8.0.mysql_aurora.3.02.0" // định nghĩa engine: mysql hay postgreSQL

  db_cluster_parameter_group_name = aws_rds_cluster_parameter_group.default.name
  db_subnet_group_name            = aws_db_subnet_group.aurora_subnet_group.name
  vpc_security_group_ids          = [aws_security_group.this.id]
  port                            = var.port

  database_name                   = var.database_name // tên của database
  master_username                 = var.master_username
  master_password                 = var.master_password

  skip_final_snapshot             = false
  final_snapshot_identifier       = "${var.app_name}-mysql-final-snapshot"
}

resource "aws_rds_cluster_instance" "this" {
  identifier              = "${var.app_name}-mysql-identifier"
  cluster_identifier      = aws_rds_cluster.this.id

  db_subnet_group_name    = aws_db_subnet_group.aurora_subnet_group.name
  db_parameter_group_name = aws_db_parameter_group.default.name

  engine                  = local.rds_engine
  instance_class          = "db.t3.medium" // định nghĩa capacity cho rds instance
}
```

Về cơ bản thì việc định nghĩa RDS không có gì quá "phức tạp", bạn đọc có thể tham khảo thêm tại: <https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/rds_cluster_instance>

Chỉ có một chú ý đó là: `master_username` và `master_password` sẽ **KHÔNG ĐƯỢC** sử dụng trong thực tế vận hành. Cũng không hẳn là không được mà là **KHÔNG NÊN**, thay vào đó:

- Mỗi một dev khi truy cập vào DB sẽ có một account riêng với các quyền được chỉ định cụ thể.
- Bản thân service khi truy cập vào DB cũng sẽ được cấp một account riêng.

Tất cả những việc trên nhằm mục đích quản lí truy xuất cũng như tracing xem **AI** đã thực thi câu SQL nào **TẠI THỜI ĐIỂM NÀO** khi hệ thống gặp sự cố hoặc bị sập.

### ecs_cluster & ecs_api modules

Tôi sẽ nói gộp 2 modules này làm một vì chúng đều liên quan đến ecs. Mục đích chính khi tôi tiến hành chia thành 2 modules con **cluster** và **api** chỉ đơn thuần là làm tách bạch phần định nghĩa **ecs_cluster** với **ecs_service** mà thôi.

Với ecs_cluster:

```terraform
resource "aws_ecs_cluster" "this" {
  name = "${var.app_name}"
}
```

ta chỉ cần định nghĩa tên của cluster là đủ.

```terraform
resource "aws_ecs_service" "this" {
  depends_on      = [aws_lb_listener_rule.this]

  name            = var.app_name

  desired_count   = 1
  launch_type     = "FARGATE"
  // sử dụng fargate với mục đích để AWS sẽ quản lí mọi server instances của service cho chúng ta

  cluster         = var.cluster_name
  task_definition = aws_ecs_task_definition.this.arn // tham chiếu đến task defintion

  network_configuration {
    subnets         = var.subnet_ids
    security_groups = [aws_security_group.this.id]
  }

  load_balancer {
    container_name   = "nginx"
    container_port   = local.port_nginx
    target_group_arn = var.lb_target_group_arn
  }
}
```

Để bạn đọc tiện theo dõi tôi sẽ đăng lại hình minh hoạ cho kiến trúc phía **ecs_service** ở đây

![Screen Shot 2023-12-07 at 22 30 17](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/7334d9cf-345d-4f41-9f90-d0496814e654)

Như bạn đọc có thể thấy

```terraform
load_balancer {
  container_name   = "nginx"
  container_port   = local.port_nginx
  target_group_arn = var.lb_target_group_arn
}
```

đoạn code này sẽ tạo ra nginx reverse proxy. Lúc này mọi request đến service sẽ đi qua nginx trước khi đi vào trong service của chúng ta.

## Tổng kết
