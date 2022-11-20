# Security In API

## Authenticate

Sử dụng các cơ chế authenticate như `OAuth`, `JWTs` hoặc `API key`.

`HTTP basic auth` là cơ chế sẽ gửi `user credential` với từng request (đây là cơ chế ít an toàn nhất).

## Validate input

Kiểm chứng đầu vào để tránh:

- SQL injection
- Cross-site scripting

Validate nên được triển khai ở 2 levels:

- `Syntactic`: theo đúng cấu trúc đã định nghĩa.
- `Semantic`: đúng kiểu giá trị và đúng giá trị quy định.

## Sử dụng API-gateway

Giải pháp `All-in-one` cho việc triển khai:

- Security
- Monitoring
- Quản lí API

Không những thế với API gateway ta cũng có thể tích hợp:

- Param validate
- Allow/Deny list
- Authen/Author
- Rate limit
- Dynamic Routing
- Service Discovery
- Error handling
- Cache
- ...

## Sử dụng Rate limiting

Rate limiting giúp bảo vệ server infra khỏi những "luồng request" mạnh (như DDoS attack chẳng hạn).

Với Rate limiting thì client sẽ rơi vào tình trạng bị "block thạm thời" nếu số lượng request gửi tới server vượt giới hạn trên cho phép.

## Only Shared required data

Đảm bảo chỉ trả về những thông tin cần thiết mà thôi, tiến hành double-check để tránh việc trả về các thông tin nhạy cảm như:

- Security
- APi keys
- ...

Cũng nên cân nhắc việc loại bỏ `X-Powered-By` trong response header để tránh leak server-side info nhằm phòng ngừa việc server sẽ bị tấn công.
