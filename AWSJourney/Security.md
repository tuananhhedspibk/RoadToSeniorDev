# Security

## KMS

- Quản lí các `secure key` theo từng region cũng như thực hiện việc mã hoá, giải mã
- Quản lí `customer master keys - (CMKs)`
- Có thể áp dụng cho: `S3 object`, `DB password`, `API keys lưu bên trong System Manager Parameter Store`

FIPS level 2

### Các loại CMK

- AWS Managed CMK
- Customer Managed CMK
- AWS owned CMK

### Symmetric vs Asymmetric CMK

**Symmetric**: cùng key cho mã hoá và giải mã - đây là loại key mặc định khi key được tạo mới

**Asymmetric**: public key/ private key (phải giữ bí mật) khác nhau

```shell
aws kms create-key --description "" # Tạo key mới

aws kms create-alias --target-key-id [key-id] --alias-name "alias/name" # Tạo alias cho key, bắt buộc phải có prefix "alias/"

aws kms list-keys

aws kms encrypt --key-id "key-id" --plaintext file://file_path --output text --query CiphertextBlob # Output cipher text blob dưới base64 encoded form

base64 --decode > output_file # Đưa kết quả base64 Ascii ra đầu ra dưới dạng binary

aws kms decrypt --ciphertext-blob fileb://file_path --output text --query Plaintext # giải mã
```

## CloudHSM

Hardware Security Module. Quản lí key của chính mình

FIPS level 3

## Parameter Store

- Thuộc về SSM (AWS Systems Manager)
- Lưu các configure hoặc secret strings như:
  - Password
  - Database connection strings
  - License codes
  - API keys

Các giá trị lưu ở đây có thể là plain text hoặc được mã hoá.

Các keys có tính kế thừa theo dạng cây kế thừa như hình bên dưới

![Screen Shot 2022-10-30 at 12 51 47](https://user-images.githubusercontent.com/15076665/198861742-71dcd5a3-5f69-4213-a833-5d8fb8dc33ae.png)

GetParametersByPath: `/dev`, `/dev/db`, ... (có thể lấy giá trị từ bất kì level nào của cây kế thừa)
