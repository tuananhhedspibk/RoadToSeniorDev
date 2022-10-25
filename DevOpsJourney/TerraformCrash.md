# Terraform with AWS crash

## Terraform là gì?

Là công cụ giúp:

- Tự động hoá việc build infrastructure
- Quản lí resources thông qua code (infrastructure as code)

## Terraform with AWS

```tf
resource "<provider>_<resource_type>" "name" {
  # config
  key = "value"
  key1 = "value"
}
```

```shell
terraform destroy # destroy resource
```

## Terraform state

Lưu trong file có đuôi là `tfstate`. File này sẽ lưu các thông tin liên quan đến infra ở thời điểm hiện tại.

File này sẽ được cập nhật mỗi khi modify hoặc thêm resource mới vào.

```shell
terraform state list
```

Lệnh trên sẽ liệt kê toàn bộ các resources hiện có
có

```shell
terraform state show resource_name
```

Lệnh trên sẽ đưa ra thông tin chi tiết về state của một resource

## Terraform output

```tf
output "server_public_ip" {
  value = "${aws_eip.ni-eip.public_ip}"
}
```

Chạy lệnh `terraform apply` sẽ đưa ra đầu ra (output) như sau:

![Screen Shot 2022-10-25 at 23 15 22](https://user-images.githubusercontent.com/15076665/197798507-53e838a8-f75b-4ad2-944d-02cd36f67472.png)

Ta có thể output bao nhiêu biến tùy ý

## Target resource

Ta sẽ thêm tuỳ chọn `-target` vào các câu lệnh `terraform destroy` hoặc `terraform apply`

```shell
terraform destroy -target resource_name
```

Chỉ gỡ bỏ đi duy nhất "resource_name"

```shell
terraform apply -target resource_name
```

Chỉ tạo duy nhất "resource_name"

## Terraform variables

```tf
variable "name" {
  description = "des"
  default = "value"
}
```

Có thể định nghĩa các biến thông qua file `.tfvars`

```shell
terraform apply -var-file file_name.tfvars
```

Sử dụng tuỳ chọn `-var-file` để chỉ định file chứa biến

Trong file `.tfvars` ta có thể khai báo object, array như sau:

```tf
subnet_info = [{ cidr_block = "10.0.1.0/24", name = "Name" }]
```

Sử dụng như sau:

```tf
subnet_info[0].cidr_block
```
