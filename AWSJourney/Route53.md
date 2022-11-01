# Route53

## DNS101

### Top level domains

Domain name được phân chia bởi dấu chấm `.`

Từ khoá cuối cùng chính là `top level domain`.

Từ khoá thứ hai sẽ là `second level domain name`.

Bản thân ELB không hề có `pre-defined IPv4`, chúng chỉ có thể resolve thông qua `DNS name`

### SOA (Start of Authority Record)

SOA record lưu các thông tin như sau:

- Tên của server cung cấp dữ liệu cho zone
- Administrator của zone
- Current version của data file
- TTL default (giây) đối với file trên resource records

### NS Record (Name server record)

Được sử dụng bởi `Top Level Domain` để từ đó đưa ra SOA chứa các DNS records như hình minh hoạ bên dưới

![File_000](https://user-images.githubusercontent.com/15076665/196028971-1da8a448-647d-4d0f-af59-f2db19ff8a45.png)

### DNS Record

### A record

A trong "A record" có nghĩa là `Address`. A record được sử dụng để chuyển từ
`domain name` thành IP address (google.com → 123.19.10.1).

Nên chọn `A record` thay vì `CNAME`

### CName

CName được sử dụng để resolve một domain name sang domain name khác.

VD: `ｍ.google.com/` sẽ dùng cho địa chỉ của web trên mobile, đồng thời với đó ta cũng muốn
`mobile.google.com/` cũng resolve địa chỉ `m.google.com/` như trên.

### Alias records

Cũng dùng để map một DNS name tới một "target" DNS name khác (có thể là ELB, Cloudfront, S3 buckets)

Tuy nhiên nếu `CName` không thể dùng cho `naked domain names` (VD: google.com) thì
`Alias Record` và `A record` lại có thể sử dụng được.

### TTL (Time to live)

Thời gian mà DNS record sẽ được cached hoặc trên `Resolving server` hoặc `user own local PC`.

Mặc định là 48 giờ.

Đơn vị của TTL là giây.

## Routing policies

### Simple Routing

![File_000 (1)](https://user-images.githubusercontent.com/15076665/196028230-c5fba4bf-abc5-4c47-a554-02aeb50792aa.png)

Các IP addresses sẽ được trả về theo một thứ tự ngẫu nhiên

### Weighted Routing

Split traffic dựa theo weighted đã assign từ trước

VD: Có thể setup `90%` traffic đi vào `ap-northeast-1` và `10%` traffic đi vào `ap-northeast-2`

### Latency-based Routing

Điều phố traffic của user dựa theo độ trễ của mạng phía user (mục đích chính là chọn ra
record với response time nhanh nhất cho user)

Như ở hình bên dưới, traffic của người dùng ở Nam Phi (South africa) sẽ được redirect đến server ở EU-West-1 thay vì AP-Southeast-2
do chênh lệch về latency

![File_000](https://user-images.githubusercontent.com/15076665/196038784-8fffefc0-1495-46a8-a9b9-7f888ffbfa3d.png)

### Failover Routing

Ta sẽ setup `active` và `passive` routing như hình bên dưới

![File_000](https://user-images.githubusercontent.com/15076665/196442482-b54f87ac-1344-4f67-b718-43864db2086c.png)

Nếu `active` routing không hoạt động thì traffic sẽ được chuyển qua `passive`

### Geolocation Routing

Cho phép điều phối traffic dựa vào vị trí địa lý của user

![File_000 (1)](https://user-images.githubusercontent.com/15076665/196444598-e4a8395f-0817-424a-9f70-0318dcc776e2.png)

### Multi Answer

Hoàn toàn tương tự như simple routing nhưng chỉ có điều ta có thể gắn `healthcheck` cho từng record.

Các records có healthcheck fail sẽ không được trả về.

### Healthcheck

Có thể gắn các records với healthcheck. Nếu record `không pass healthcheck` nó sẽ bị `loại bỏ khỏi Route53`
cho đến khi pass healthcheck thì thôi
