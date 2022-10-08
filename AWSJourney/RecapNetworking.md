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

