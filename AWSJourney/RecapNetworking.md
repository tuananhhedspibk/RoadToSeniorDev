# Recap Networking Knowledge

Tham khảo từ:

- <https://www.engisv.info/?p=350>
-

## IPv4 address

Địa chỉ IPv4 gồm 32 bit, có thể biểu diễn thành 4 `octets` (mỗi octect sẽ bao gồm 8 bits)

VD: 192.168.136.28 = 11000000 10101000 10001000 00011100

Địa chỉ IP được chia thành 2 phần `NetworkID`, `HostID`:

- `NetworkID` dùng để định tuyến trên đường mạng
- `HostID` dùng để định danh cho từng host (máy)

![IPv4 phan 1 (1)](https://user-images.githubusercontent.com/15076665/194710188-48975a20-3f90-4273-942a-4642e24cb11e.png)

Có 2 địa chỉ đặc biệt là:

- `Địa chỉ mạng`: Toàn bộ bit trong host ID đều bằng 0 (đại diện cho 1 mạng, các host trong mạng đi ra ngoài internet sẽ sử dụng địa chỉ này)
- `Địa chỉ broadcast`: Toàn bộ bit trong host ID đều bằng 1 (đại diện cho toàn bộ hosts trong mạng, khi gửi mess đến địa chỉ này thì mess sẽ được gửi đến toàn bộ các hosts trong mạng)

## Các lớp địa chỉ IP

### Lớp A

Bit đầu tiên trong `NetworkID` bằng `0`, sử dụng octet đầu tiên làm network ID.

![IPv4 phan 1 (3)](https://user-images.githubusercontent.com/15076665/194710824-7696172f-3467-4cd0-b38e-e5dbadac71d8.png)

Về lí thuyết các địa chỉ mạng sẽ đi từ `0.0.0.0` đến `127.0.0.0`.
Nhưng `0.0.0.0` là địa chỉ mạng của các mạng, nó sẽ đại diện cho tất cả các mạng trong LAN

`127.0.0.0` là địa chỉ `loopback`

### Lớp B

Sử dụng 2 octets đầu tiên cho network ID nhưng:

- Bit đầu tiên luôn là 1
- Bit thứ hai luôn là 0

![IPv4 phan 1 (4)](https://user-images.githubusercontent.com/15076665/194758554-bf363d51-90f1-46d0-8a83-f3c4a10678a0.png)

Nên chỉ có 16,384 mạng (128.x.y.z - 191.x.y.z), mỗi mạng trong lớp B sẽ có hơn 65,000 host.

### Lớp C

Sử dụng 3 octets đầu tiên làm network ID. Ba bits đầu lần lượt là: 1, 1, 0

![IPv4 phan 1 (5)](https://user-images.githubusercontent.com/15076665/194758722-dfa9ad35-8679-46b6-baa3-455d8e080dee.png)

Mỗi mạng chỉ có 254 hosts. Dải địa chỉ mạng sẽ đi từ `192.x.y.z` - `223.x.y.z`

## IP public & private

IP private sinh ra để dùng cho các mạng nội bộ (LAN) do sự phát triển mạnh mẽ của mạng nội bộ nên nếu mỗi thiết bị đều có một địa chỉ IP thì sẽ dẫn đến việc không đủ địa chỉ IP để cấp phát cho các thiết bị.

Các IP privates tương ứng với từng lớp là như sau:

![IPv4 phan 1 (7)](https://user-images.githubusercontent.com/15076665/194758898-9c8b97ae-db4f-4c7e-ae7d-62b19a848624.png)

Các thiết bị trong mạng LAN đi ra ngoài thông qua NAT (network address translation). NAT được cung cấp trên:

- Firewall
- Router

Các hosts trong mạng LAN sẽ sử dụng chung IP public của từng mạng

![File_000](https://user-images.githubusercontent.com/15076665/194759263-39e35cae-7061-4153-bcce-11f22fe27de1.png)

## Mặt nạ mạng (Subnet mask)

Dùng để xác định địa chỉ mạng của một địa chỉ IP bất kì. Khi đó các bit thuộc network ID sẽ được bật lên thành 1

![subnet-1](https://user-images.githubusercontent.com/15076665/194759483-712ebc62-9b06-4a3a-b7d3-ed24a9ba157e.png)

VD: với địa chỉ `192.168.1.2` - đây là địa chỉ thuộc lớp C, nó có subnet mask là `255.255.255.0`, nên địa chỉ mạng của nó là: `192.168.1.0`, địa chỉ IP trên cũng có thể được viết thành `192.168.1.2/24` (/24 là số bits của network ID).

Subnet mask như trên còn được gọi là `default subnet mask`. Bên cạnh nó ta còn có thể chia nhỏ mạng thành các subnets với 2 mục đích sau:

- Tiết kiệm IPs: như đã biết mỗi mạng thuộc lớp B có thể chứa hơn 65,000 hosts, mỗi mạng thuộc lớp C chỉ có thể chứa 254 hosts, nên nếu trong trường hợp mạng của chúng ta có chừng 300 hosts thì việc sử dụng mạng lớp B sẽ lãng phí cả chục ngàn host ID, nên ta sẽ tiến hành chia mạng thuộc lớp B này thành các subnets để tiết kiệm IP
- Chia nhỏ mạng con khi trong mạng có một lượng lớn hosts (có thể khiến hạ tầng mạng quá tải)

## Chia mạng con

Bản chất của việc chia mạng con là mượn thêm các bits ở phần `host ID` cho `network ID`

![subnet-2](https://user-images.githubusercontent.com/15076665/194760569-0e6ae1ec-1c82-41d5-9a2e-de58193b171d.png)

VD: với địa chỉ IP: `144.28.0.0` - thuộc lớp B, ta vay thêm `4 bits` ở host ID, lúc này sẽ có 20 bits dùng để làm `network ID`. Subnet mask là: `255.255.240.0`

Một điều cần lưu ý đó là mạng con luôn thuộc lớp cha của nó, ví dụ với `10.0.0.0/16` vẫn thuộc lớp A dù là `/16`

Với địa chỉ `144.28.16.17/20`, ta sẽ xác định địa chỉ mạng của nó như sau:

144.28.16.17  = 10010000 00011100 00010000 00010001 (IP address)

AND logic

255.255.240.0 = 11111111 11111111 11110000 00000000 (Subnet mask)

Ta có kết quả network ID là:

144.28.16.0   = 10010000 00011100 00010000 00000000
