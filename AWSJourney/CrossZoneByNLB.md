# Cross Zone Load Balancing bằng Network Load Balancer

Bài viết được dịnh từ [nguồn](https://dev.classmethod.jp/articles/cross-zone-load-balancing-for-nlb/)

## Cross Zone Load balancing

Tính năng Cross-Zone Load balancing đã được AWS Network Load Balancer hỗ trợ từ năm 2018.

Với cùng một địa chỉ IP, NLB có thể phân tải đến tất cả các instances phía dưới nó dù không cùng nằm trong một AZ. Điều này sẽ đem lại những điểm lợi sau:

- Tăng tính khả dụng của địa chỉ IP
- Phân tải đều đến tất cả các Zones

### Tăng tính khả dụng của địa chỉ IP

Như hình minh hoạ bên dưới, ta có 2 Zones, mỗi Zone có một instance.

![NLB-Non-CrossZone](https://user-images.githubusercontent.com/15076665/209677666-c1804859-7751-4a3b-93f9-3c9862cbdd1a.png)

Trong trường hợp có một instance gặp sự cố

![NLB-Non-CrossZone1](https://user-images.githubusercontent.com/15076665/209677794-91be5c7c-1794-44d9-b2e6-6e0e31d4d8cf.png)

Do instance không thể hồi đáp nên client sẽ coi như phía bên phải của NLB là không thể kết nối được.

![NLB-Non-CrossZone2](https://user-images.githubusercontent.com/15076665/209677923-d6051239-53b8-4491-b06b-b640f1f26a4a.png)

Do NLB có khả năng cố định địa chỉ IP nên nó có thể thiết lập TTL khá dài cho DNS, từ đó dẫn đến tình trạng nếu một instance bị "sập" thì địa chỉ IP cố định đó sẽ không thể truy cập được nữa.

Tiếp theo, hãy thử nghĩ theo hướng cross-zone.

![NLB-CrossZone](https://user-images.githubusercontent.com/15076665/209680269-3c12f5b5-6140-4a17-bf1e-2e8a14d89a19.png)

Giả sử trong trường hợp instance phía bên phải không truy cập được, NLB sẽ coi instance bên phải là `Unhealthy`, và toàn bộ traffic tới địa chỉ IP cố định sẽ được chuyển tới instance phía bên trái như hình bên dưới.

![NLB-Non-CrossZone3](https://user-images.githubusercontent.com/15076665/209680469-42fee28c-b291-478e-a40d-b31721a0618f.png)

Nhờ đó, kể cả khi số lượng instance bị giảm đi thì các địa chỉ IP vẫn luôn luôn ở trạng thái "khả dụng", tuy nhiên lúc này do số lượng instance giảm đi nên lượng tải mà instance phải chịu cũng sẽ tăng lên.

### Cân bằng tải giữa các zones

Tiếp theo ta sẽ xét đến trường hợp 4 instances được khởi động, ta có 2AZ, tương ứng với mỗi AZ sẽ là 2 instances.

![NLB-Non-CrossZone-1](https://user-images.githubusercontent.com/15076665/209753048-62c643a6-c7b4-40de-9291-68ebfcf8b8f0.png)

Tuy nhiên vẫn có thể có trường hợp phân bổ các EC2 instances giữa các AZs bị lệch như hình bên dưới:

![NLB-Non-CrossZone-2](https://user-images.githubusercontent.com/15076665/209753247-1774cd93-dc9a-4b14-a715-fddc538a9d6c.png)

Do NLB sẽ phân bổ tải cân bằng đến các instances nên mô hình chung ta sẽ có phân bổ tải như sau: 1/2 & 1/6, 1/6, 1/6 - hình minh hoạ bên dưới

![NLB-Non-CrossZone-2-2](https://user-images.githubusercontent.com/15076665/209753334-55180f6c-48b8-4ba8-912b-2c62c22a9990.png)

Với cross-zone balacing, NLB sẽ tiến hành phân bổ tải đều lên các instances do đó các instance sẽ được sử dụng đúng và đủ công suất.

![NLB-CrossZone-2-3](https://user-images.githubusercontent.com/15076665/209753582-8e19ad82-030e-4d32-a228-644fdb2b5e0c.png)

## Chú ý

