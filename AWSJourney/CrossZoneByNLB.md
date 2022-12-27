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
