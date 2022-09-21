# Một vài chia sẻ về SocketIO Emitter

Trong thời gian gần đây tôi có tiếp xúc trực tiếp với `Socket.io` trong công việc của mình. Bài toán của tôi có thể diễn giải đơn giản như sau:

Với 2 clients cho trước (1 Web và 1 App), làm cách nào để `client 1 - App` có thể gửi dữ liệu cho `client 2 - Web` để xử lí. Theo như hiểu biết cá nhân thì tôi thấy rằng không có cách nào để `client 1` có thể gửi trực tiếp dữ liệu cho `client 2` cả. Chỉ có một cách duy nhất đó là thông qua 1 server trung gian ở giữa đóng vai trò "trung chuyển" dữ liệu như hình mô tả dưới đây

![File_000](https://user-images.githubusercontent.com/15076665/191404991-c978e598-782e-4f00-9048-f303aad650bb.png)

OK, vậy là bài toán đã được xác định, lúc này hướng giải quyết là như thế nào ?

Với tương tác từ `client 1` đến `server` ta có thể sử dụng lời gọi API thông thường được, nhưng tương tác từ `server` đến `client 2` thì **không thể** sử dụng "lời gọi API được". Theo lí thuyết ta có thể sử dụng một trong 3 phương thức dưới đây:

- Polling
- Long Polling
- Web socket

Với `Polling` thì như hình bên dưới client sẽ "liên tục hỏi" server về việc có dữ liệu mới hay không ?. Tần suất hỏi này được thiết lập dựa theo "tần suất polling". Trong trường hợp câu trả lời là **không** liên tục thì việc "hỏi liên tục" này sẽ gây **lãng phí thời gian** và **lãng phí tài nguyên**

![Screen Shot 2022-09-15 at 23 26 56](https://user-images.githubusercontent.com/15076665/190430305-af1c63c2-4105-47d2-871e-6a34f8ba594b.png)

Do `Polling` hoạt động "không hiệu quả" nên ta có một cách tiếp cận khác là `Long Polling`, như hình mô tả bên dưới client sẽ "giữ" cho connection tồn tại cho đến khi:

- Nhận được data trả về từ server
- Đạt tới ngưỡng `timeout`

Khi client nhận được data trả về từ server thì nó sẽ gửi request đến server để tiếp tục quá trình `Long Polling` như trên. Tuy vậy `Long Polling` vẫn có những nhược điểm như sau:

- `client 1` và `client 2` ở trong bài toán phía trên "có thể" sẽ không kết nối tới cùng một server (đây cũng là một vấn đề mà tôi gặp phải trong quá trình dev, vấn đề này sẽ được trình bày ở phần kế). Nguyên nhân do HTTP là `stateless` - tức là server sẽ không biết gì về trạng thái của client, nên nếu `load balancer` sử dụng giải thuật `round robin` để phân bổ request thì "có thể" server nhận được data không kết nối được với client cần nhận data
- Server cũng không thể báo cho client biết được khi kết nối bị ngắt
- Trong trường hợp data không được gửi quá nhiều thì `request long polling` vẫn được gửi liên tục khi `timeout`

![Screen Shot 2022-09-16 at 8 14 56](https://user-images.githubusercontent.com/15076665/190525180-b8ba06b5-28e3-40f6-a230-58e2b129de6c.png)

Cách tiếp cận cuối cùng ở đây là `web socket`, đây là một cách tiếp cận phổ biến giúp server có thể cập nhật các sự thay đổi cho client.

![Screen Shot 2022-09-17 at 10 46 43](https://user-images.githubusercontent.com/15076665/190835781-49d79114-6b27-4df5-8d27-4b0b74d945f2.png)

Nói qua thì WebSocket là một kết nối hai chiều (bi-directional) được khởi tạo phía client. Khi khởi tạo thì nó vẫn là `HTTP connection` nhưng sẽ được "upgrade" để trở thành `WebSocket connection` thông qua các bước "handshake". Ở đây tôi không tiện nói sâu về quá trình này nhưng bạn có thể hiểu nôm na như sau:

B1: Một kết nối HTTP/1.1 vẫn sẽ được tạo ra giữa client và server

B2: Client sẽ gửi một "upgrade" HTTP request với `header` bao gồm tối thiểu 2 thông tin sau

```JSON
Connection: Upgrade
Upgrade: websocket
```

B3: Client chờ đến khi server reply lại response có chứa `HTTP 101 Switching Protocols`, response này cho thấy server đang chuyển sang protocol mà client yêu cầu upgrade trong header của request gửi lên
B4: Sau khi nhận được response thì WebSocket connection sẽ được thiết lập

Chi tiết hơn về WebSocket các bạn có thể tìm đọc ở 2 link bên dưới:

- <https://sookocheff.com/post/networking/how-do-websockets-work/>
- <https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism>

Trở lại với vấn đề của chúng ta, với WebSocket connection "bền vững" này server hoàn toàn có thể cập nhật mọi thay đổi cho phía client đồng thời việc quản lí kết nối cũng sẽ nằm hoàn toàn ở phía server

*Chú thích*: những phần về `Polling`, `Long Polling`, `Web socket` tôi đều tham khảo từ sách `System Design Interview - P1` của tác giả `Alex Xu`. Các bạn có thể tìm và mua tại [link](https://www.amazon.co.jp/System-Design-Interview-insiders-Second/dp/B08CMF2CQF/ref=asc_df_B08CMF2CQF/?tag=jpgo-22&linkCode=df0&hvadid=453560710311&hvpos=&hvnetw=g&hvrand=2259595574916540920&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=1009309&hvtargid=pla-934212337151&psc=1&th=1&psc=1)

Với bài toán của mình tôi quyết định sử dụng `socket.io`. Nói qua cho những bạn chưa biết thì `socket.io` là một thư viện khá nổi tiếng để triển khai `WebSocket connection` cho các dự án `Realtime chat`, ...

Trang chủ của nó là: <https://socket.io/>

Dự án tôi đảm nhận có sử dụng `NestJS` (<https://nestjs.com/>) cho server và `ReactJS` (<https://reactjs.org/>) cho phía web (client 2).

Các bạn có thể tham khảo code minh hoạ ở repository sau: <https://github.com/tuananhhedspibk/SocketIoEmitterBlog>
