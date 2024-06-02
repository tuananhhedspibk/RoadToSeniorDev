# CloudFront

## Fundamental

- CDN
- Content được cache ở edge
- DDoS protection

## Key Terminology

**Edge location**: nơi mà content sẽ được cached

**Origin**: origin of file, có thể là S3 bucket, EC2 instance hoặc ELB

**Distribution**: tên của CDN bao gồm nhiều Edge locations

![File_000](https://user-images.githubusercontent.com/15076665/198866865-002bb500-e220-4015-bef7-f326c803701f.png)

**Web Distribution**: sử dụng cho web

**RTMP**: sử dụng cho media streaming

Object có thể được cache trong TTL tại Edge location.

## Signed URLs & Cookies

Signed URL dùng cho `1 file duy nhất`

Signed Cookie dùng cho `nhiều files`

Signed URL, Cookie đều cần các policies với:

- URL expiration
- IP ranges
- Trust signers

Với cloudfront signed URL client không thể trực tiếp truy cập tới `origin` giống như S3 signed URL

![File_000 (1)](https://user-images.githubusercontent.com/15076665/198867217-be6a0c01-af31-4bef-9462-54a2c9bf04fe.png)

## Athena vs Macie

### Athena

Interactive query service cho phép truy vấn dữ liệu từ S3 dưới dạng SQL (serverless)

Dùng để query log file trong S3, access log trong S3

### Macie

PII (Personally identifiable information)

Macie sử dụng NLP, ML để tìm, phân loại cũng như bảo vệ các dữ liệu nhạy cảm trong S3
