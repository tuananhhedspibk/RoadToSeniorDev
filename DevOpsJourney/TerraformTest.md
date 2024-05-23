# Ghi chép chuẩn bị cho Terrafor Associate Test

## terraform state

Chỉnh sửa state, ví dụ như loại bỏ items, ...

## terraform force-unlock

Bẻ khoá (lock) state hiện thời

```sh
terrform force-unlock LOCK_ID
```

## Enabled logging

Set `TF_LOG` env variable để cho phép `detailed log`

## terraform console

Launch interactive console của terraform.

## terraform validate

Câu lệnh này dùng để kiểm tra và báo cáo lỗi trong module, attribute names, value types để đảm bảo chúng đúng với cú pháp cũng như đảm bảo sự thống nhất bên trong.

## terraform workspace select [name]

Chuyển giữa các workspaces. Về cơ bản workspace cho phép chúng ta `dùng một config cho nhiều môi trường (dev, stg, prod)` nhưng state của các môi trường này là riêng biệt.

Sau khi khởi tạo, chúng ta sẽ có `default workspace`

## terraform workspace new [name]

Tạo mới một workspace.

## terraform workspace list

Danh sách các workspace

## terraform workspace delete [name]

Xoá workspace nhưng chú ý rằng ta không thể xoá `default workspace`.

```tf
provider "aws" {
  region = var.region
}

resource "aws_instance" "example" {
  count         = terraform.workspace == "production" ? 5 : 1
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  tags = {
    Name = "example-${terraform.workspace}"
  }
}
```

## State file của workspace

Được lưu trong `terraform.tfstate.d/[workspace_name]`

## Implicit Dependencies

Đây là cách terraform tạo ra dependency graph từ config file dựa theo thứ tự tạo cũng như tham chiếu giữa các resources.

```tf
resource "aws_security_group" "example" {
  name        = "example-sg"
  description = "Example security group"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  security_groups = [
    aws_security_group.example.name
  ]

  tags = {
    Name = "example-instance"
  }
}
```

Như ở ví dụ trên, muốn tạo `aws_instance` cần phải có `aws_security_group` nên `security_group` sẽ được tạo trước.
`security_group` sẽ được gọi là `implicit dependency`

### Lợi ích

1. Đơn giản hoá config khi không cần dùng từ khoá `depends_on`
2. Đảm bảo resource được tạo đúng thứ tự

## Explicit Dependencies

```tf
resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  depends_on = [
    aws_security_group.example
  ]

  tags = {
    Name = "example-instance"
  }
}
```

Với việc sử dụng từ khoá `depends_on` ta có thể thấy một cách "tường minh" rằng `aws_instance` sẽ phụ thuộc vào `aws_security_group`

## terraform apply -replace=[Name]

Tạo lại resource

```tf
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  tags = {
    Name = "example-instance"
  }
}
```

```sh
terraform apply -replace=aws_instance.example
```

## alias

Sử dụng `alias` để có thể sử dụng **multiple provider blocks với cùng kiểu**

```tf
provider "aws" {
  region = "us-west-1"
  alias  = "us_west_1"
}

provider "aws" {
  region = "us-east-1"
  alias  = "us_east_1"
}

resource "aws_instance" "web_us_west_1" {
  provider = aws.us_west_1
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  tags = {
    Name = "web-us-west-1"
  }
}

resource "aws_instance" "web_us_east_1" {
  provider = aws.us_east_1
  ami           = "ami-87654321"
  instance_type = "t2.micro"

  tags = {
    Name = "web-us-east-1"
  }
}
```

## terraform plan -refresh-only

Chỉ cập nhật state file mà không làm thay đổi gì đến infra của provider. Thường dùng trong trường hợp khi resource online bị chỉnh sửa bằng tay, ta sẽ chạy lệnh này để cập nhật lại state file.

## Chỉ destroy một phần resource

Bước 1: Chạy `terraform state rm` để loại bỏ đi resource mà ta **KHÔNG MUỐN DESTROY** khỏi state.

Bước 2: Chạy `terraform destroy`

## terraform provision

Mặc định `terraform apply` có thể provision lên đến **10 resources** (quá trình provision được thực thi một cách song song) - đồng nghĩa với việc terraform có thể **cùng một lúc** thêm mới, cập nhật, xoá 10 resources.

Con số này được thiết lập dựa theo `-parallelism` flag.

Thường điều chỉnh con số `parallelism` theo các trường hợp như sau:

- Deploy hệ thống lớn
- CI/CD pipeline (do yêu cầu thời gian nghiêm ngặt)
- API Rate limit (do API limit từ phía provider)

## Yêu cầu để publish terraform public registry

1. Cần có github public repo tương ứng
2. Repo name phải theo format `terraform-<PROVIDER>-<NAME>` - với provider = aws hoặc google, ... name = tên mô tả repo (VD: terraform-aws-ec2-instance)
3. Module structure

```css
├── main.tf
├── variables.tf
├── outputs.tf
└── README.md
```

4. Document phải gồm các phần:

- Description
- Usage
- Inputs
- Outputs

5. Semantic versioning

Tag version trên github repo phải theo semantic version (`v1.0.0`, `v1.0.1`)

```sh
git tag v1.0.0
git push origin v1.0.0
```

## terraform import

Để import các resources có sẵn vào terraform state. Cho phép ta có thể đưa các resources không nhất thiết được tạo bằng terraform vào sự quản lí của terraform mà không cần phải tạo lại resources.

```sh
terraform import ADDRESS ID
```
