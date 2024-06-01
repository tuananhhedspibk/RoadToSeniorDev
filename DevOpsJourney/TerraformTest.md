# Ghi chép chuẩn bị cho Terrafor Associate Test

## State trong terraform

Thường sẽ lưu thông tin liên quan đến resources:

- Resource ID
- Resource attributes
- ...

lưu dưới dạng plain-text JSON. Các thông tin "nhạy cảm" như `database password` hay `database username` nên được lưu tại các `remote backend`.

State sẽ cache lại các attribute values nên giúp cải thiện về mặt hiệu năng. Khi chạy `terraform plan` cho các infrastructure với quy mô lớn, việc query các resource sẽ gặp các vấn đề như sau:

- Chậm
- Nhiều cloud provider không cung cấp API để query nhiều resources một lúc.
- Cloud Provider API Rate limitter.

nên việc cache các thông tin về resource trong state sẽ giúp cải thiện đáng kể về mặt hiệu năng. Ngoài ra ta có thể sử dụng `terraform plan -refresh=false` để tận dụng cache state.

### Migration

Ta có thể chuyển (migrate) state từ backend này sang backend khác.

```sh
terraform init -migrate-state
```

Lệnh này sẽ giúp tái sử dụng lại các providers được đọc từ lock file.

[Screenshot 2024-05-25 at 22 14 17](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/b6254670-7004-4597-a981-877f3ee1d10e)

## Câu lệnh liên quan đến terraform

### terraform console

Launch interactive console của terraform.

Lệnh này đọc terraform configure ở working dir hiện tại và terraform state file nên CLI phải có khả năng lock state để tránh sự thay đổi state.

### terraform force-unlock

Bẻ khoá (lock) state hiện thời

```sh
terrform force-unlock LOCK_ID
```

Terraform "lock" state để tránh các concurrent modifies có thể gây ra lỗi.

Chỉ có một vài backend hỗ trợ locking:

- GCP
- Azure
- AWS S3 with DynamoDB

### terraform state

Chỉnh sửa state, ví dụ như loại bỏ items, ...

### terraform show

Inspect state

### terraform init

`terraform init -backend-config=PATH`: chỉ định file config (các files này sẽ chứa thông tin nhạy cảm nên được giữ trong các secure data store).

`terraform init -backend-config="KEY=VALUE"`: command line key/value pairs.

`terraform init` cũng caches source code local cho các modules tương ứng.

### terraform validate

Câu lệnh này dùng để kiểm tra và báo cáo lỗi trong module, attribute names, value types để đảm bảo chúng đúng với cú pháp cũng như đảm bảo sự thống nhất bên trong.

### terraform apply

Chú ý rằng khi chạy lệnh apply, ngay cả khi có những resources bị rollback thì những resources nào được tạo thành công vẫn sẽ được deployed mà không hề bị rollback.

#### Với config file (tf) rỗng

Toàn bộ resources sẽ bị huỷ.

#### -replace=[Name]

Tạo lại resource (forcing). Thường chỉ nên sử dụng lệnh này khi resource gặp vấn đề **không liên quan đến terraform**

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

### terraform refresh

Đọc toàn bộ các settings từ các remote objects và cập nhật lại state để match với remote objects.

Đây chính là bước đầu tiên được chạy khi thực thi lệnh `terraform plan`

`terraform plan -refresh-only`: Chỉ cập nhật state file mà không làm thay đổi gì đến infra của provider. Thường dùng trong trường hợp khi resource online bị chỉnh sửa bằng tay, ta sẽ chạy lệnh này để cập nhật lại state file. Do `terraform refesh` bị deprecated nên hãy sử dụng `terraform plan -refresh-only` với cùng tác dụng như nhau.

### terraform import

Để import các resources có sẵn vào terraform state. Cho phép ta có thể đưa các resources không nhất thiết được tạo bằng terraform vào sự quản lí của terraform mà không cần phải tạo lại resources.

```sh
terraform import ADDRESS ID
```

### terraform get

Dùng để download module từ registry hoặc version control system.

Lệnh này có thể được dùng để cập nhật version của một module cụ thể tới một version nhất định nào đó.

Nếu cần cập nhật module cụ thể thì nên dùng lệnh này thay vì `terraform init`

### terraform fmt

Format terraform config file. Đảm bảo codebases có tính thống nhất cao.

`-recursive`: chạy với cả subdirectories.

### terraform plan

`-out` cho phép chúng ta save plan để apply lúc sau.

### terraform graph

Đưa ra đồ thị dependency trong config hiện thời dưới dạng ngôn ngữ `DOT`. Ngoài ra nó cũng có thể visual config hoặc execute plan.

## Terraform Log

### Enabled logging

Set `TF_LOG` env variable để cho phép `detailed log`

Set `TF_LOG_PATH` để thiết lập đường dẫn cho file lưu log.

## Terraform Workspace

### terraform workspace select [name]

Chuyển giữa các workspaces. Về cơ bản workspace cho phép chúng ta `dùng một config cho nhiều môi trường (dev, stg, prod)` nhưng state của các môi trường này là riêng biệt.

Sau khi khởi tạo, chúng ta sẽ có `default workspace`

### terraform workspace new [name]

Tạo mới một workspace.

### terraform workspace list

Danh sách các workspace

### terraform workspace delete [name]

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

### State file của workspace

Được lưu trong `terraform.tfstate.d/[workspace_name]`

## Dependencies

### Implicit Dependencies

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

#### Lợi ích

1. Đơn giản hoá config khi không cần dùng từ khoá `depends_on`
2. Đảm bảo resource được tạo đúng thứ tự

### Explicit Dependencies

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

### Resource dependencies

Terraform sẽ phân tích config trong terraform để tìm ra tham chiếu tới các resources khác. Từ đó đưa ra thứ tự `CRUD` các resources một các chính xác nhất.

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

### remote-exec & local-exec

Là các provisioner được sử dụng để chạy các scripts hoặc command trên môi trường lần lượt là `remote` và `local`.

Thường dùng trong quá trình boostrapping, config resource.

Ví dụ với `remote-exec`

```tf
resource "aws_instace" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  provisioner "remote-exec" {
    connection {
      type = "ssh"
      user = "ec2-user"
      private_key = file("~/.ssh/my-key.pem")
    }

    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo service nginx start"
    ]
  }
}
```

Ví dụ với `local-exec`

```tf
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"

  provisioner "local-exec" {
    command = "echo 'S3 bucket created!' > /tmp/s3_notification.txt"
  }
}
```

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

## Module

```tf
module "vault-aws-tgw" {
  source  = "terraform-vault-aws-tgw/hcp" // source chỉ ra public registry mà module tham chiếu tới
  version = "1.0.0"

  client_id      = "4djlsn29sdnjk2btk"
  hvn_id         = "a4c9357ead4de"
  route_table_id = "rtb-a221958bc5892eade331"
}
```

## Backend

Backend trong terraform là config chỉ ra rằng state được loaded, lưu trữ như thế nào.

Các loại backend types:

- `local`: lưu state file trong local file. Đây là default backend.
- `consul`: là một sự lựa chọn phổ biến để lưu Terraform state.
- `s3`: lưu terraform state trên Amazon S3.

### Tại sao lại sử dụng backend

1. `Collab`: tránh việc phát sinh conflict khi nhiều members cùng làm việc trên cùng một môi trường infra.
2. `State locking`: tránh việc đồng thời sửa state cùng một lúc.
3. `Remote storage`: đảm bảo state file được lưu ở nơi tập trung và có tính bảo mật cao.
4. `Backup, Recovery`: tự động backup và recovery.

## TF_VAR

Đây là prefix string dùng cho thiết lập input variables sử dụng biến môi trường trong terraform.

```sh
# export biến môi trường.
export TF_VAR_instructor_name="bryan"

# Apply, sử dụng biến môi trường.
terraform apply
```

## Kiểu ngôn ngữ của terraform

1. Immutable
2. Declarative IaC provisiong language
3. Dựa trên HCL hoặc JSON cho config files.

## Sensitive data

Các thông tin nhạy cảm như `key` hoặc `database password`, ...

Ta có thể sử dụng backend như `S3` để mã hoá state cũng như các thông tin nhạy cảm.

State best pratice:

1. Coi state như `sensitive data`.
2. Mã hoá state backend.
3. Control access tới state file.

### State được lưu dưới dạng plain text

Ngay cả với các thông tin được đánh dấu là `sensitive`. Terraform run có thể in các giá trị "nhạy cảm" này ra log.

### Terraform vault

Là công cụ quản lí và bảo vệ các sensitive data. SỬ dụng `Vault` provider từ terraform.

```tf
// Khai báo provider
provider "vault" {
  address = "https://vault.example.com:8200"

  token = "s.your-vault-token"
}

// Ghi secret vào vault
resource "vault_generic_secret" "example" {
  path = "secret/data/myapp/config"

  data_json = jsondecode({
    username = "admin"
    password = "s3cr3t"
  })
}

data "vault_generic_secret" "example" {
  path = "secret/data/myapp/config"
}

output "username" {
  value = data.vault_generic_secret.example.data["secret/data/myapp/config"]
}
```

Một lưu ý là với các thông tin nhạy cảm như:

- Vault tokens
- Vault role IDs
- Vault secret IDs

ta cần lưu chúng vào các biến môi trường.

```sh
export VAULT_ADDR='https://vault.example.com:8200'
export VAULT_TOKEN='s.your-vault-token'
```

Việc sử dụng vault có nhược điểm đó là các `secrets` sẽ được lưu trong `state file`. Do đó các `state file` này sẽ được coi như chứa các thông tin "nhạy cảm" và cần được bảo vệ.

## Credential Information

Ta có thể sử dụng `AWS IAM` hoặc `Azure Managed Service Identity` để cung cấp credential cho terraform. Với các services này ta có thể kết nối tới API mà không cần expose các thông tin nhạy cảm trong terraform config.

## Verbose logging terraform

Để enable verbose logging trong terraform, ta cần thiết lập giá trị cho biến môi trường `TF_LOG` với các level logs như sau:

- `TRACE`: show các internal logs của terraform. Cho ta thấy được nhiều thông tin chi tiết hơn (các bước đi từ `terraform plan`, `terraform apply` hay `terraform destroy`)
- `DEBUG`
- `INFO`
- `WARN`
- `ERROR`

## Terraform block

Định nghĩa settings và behaviors cụ thể.

```tf
terraform {
  required_version = ">= 0.12.0"

  backend "s3" {
    bucket = "my-terraform-state"
    key    = "terraform.tfstate"
    region = "us-west-2"
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "3.57.0"
    }
  }
}
```

`required_providers` được dùng để chỉ ra version cụ thể của provider mà ta muốn sử dụng.

## Các vấn đề của việc quản lí Infra bằng tay

1. Khó đảm bảo các components được config một cách thống nhất.
2. Khó đảm bảo sự thống nhất, tiêu chuẩn giữa các môi trường khác nhau.
3. Provisioning chậm.
4. Khó tránh được các lỗi liên quan đến con người như `missed config`, ...
5. Khó trong việc tạo document.

## Terraform provider

Offical terraform providers và modules đều được sở hữu và maintaince bởi Hashicorp.

Provider có thể được terraform tự động detect và download khi chạy lệnh `terraform init`.

Provider có thể được cài qua:

- Terraform registry.
- Terraform plugin cache.
- Plugins directory.

## Terraform Cloud

## Mối liên hệ với Terraform CLI

![Screenshot 2024-05-30 at 21 42 59](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/e785b0b6-9890-458a-b5f5-34c0b1644308)

### Workspace

Workspace trong terrraform cloud sẽ là các working directories riêng biệt chứ không giống như workspace của `terraform workspace`

Naming convention `a-b-c`.

![Screenshot 2024-05-30 at 21 44 26](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/1419d30c-2cb2-4d7f-acb1-fe0c90bbf1bf)

### Authentication

`terraform login` sẽ sử dụng `API Token` từ `app.terraform.io` để login vào terraform cloud.

### Secure variables

Có thể chuyển sang chế độ sensitive với các thông tin quan trọng như key, ...

### Version control

Mục đích chính là để nhiều người có thể làm việc trên cùng một terraform code.

Được tích hợp với các version control như `Github`, `Bitbucket`, ...

### Private registry

Lưu và versioning các terraform modules cho mục đích tái sử dụng.

Cũng có thể liên kết từ Github (VCS) repo sang. Các repo này phải có tên theo format `terraform-<PROVIDER>-<NAME>`

### Sentinel policy

Có thể hiểu như là Policy-as-code. Sentinel policy sẽ nằm ở giữa `terrform plan` và `terrform apply` stages.

Policy có 3 levels chính đó là:

- `Advisory`: policy được cho phép failed, sẽ đưa ra log ở dạng warning.
- `Soft Mandatory`: cho phép override còn không thì phải pass.
- `Hard Mandatory`: bắt buộc phải pass, không cho phép override.

Sentinal policy evaluation sẽ được thực thi `sau khi terrảom hoàn thành plan, cost estimation và trước khi apply`.

### Version control workflow

Dùng khi làm việc nhóm. Liên kết terraform workspace với VCS repo và trigger apply hoặc plan mỗi khi commit code.

Ngoài ra nhiều workspace (dev, stg, prod) có thể liên kết và sử dụng chung một code base từ VCS repo.

Với các workspace liên kết với VCS, yêu cầu VCS-driven workflow để đảm bảo `single source of truth` chứ không được phép chạy `terraform apply` bằng CLI hoặc phương thức khác (vẫn có thể chạy `terraform plan`).

### Agents

Thực thi terraform plan và apply changes tới infrastructre

Chức năng cơ bản:

- Nhận `terraform plan` từ Terraform cloud.
- Chạy plan dưới local.
- Apply những sự thay đổi đó.

## Infrastructure as code

Có khả năng:

- Versionning cho infra.
- Chia sẻ, tái sử dụng code.
- Tạo blueprint cho data center.

Cho phép `API-driven workflow` cho việc deploy resource trong public cloud, private infra, ...
