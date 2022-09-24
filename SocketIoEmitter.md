# Một vài chia sẻ về nestjs, socket.io và @socket.io/redis-adapter

## Đặt vấn đề

Trong thời gian gần đây tôi có tiếp xúc trực tiếp với `socket.io` trong công việc và gặp một vấn đề liên quan đến việc truyền dữ liệu giữa các clients thông qua socket. Vấn đề của tôi có thể diễn giải đơn giản như sau:

Với 2 clients cho trước (1 Web và 1 App), làm cách nào để `client 1 - App` có thể gửi dữ liệu cho `client 2 - Web` để xử lí. Theo như hiểu biết cá nhân thì tôi thấy rằng không có cách nào để `client 1` có thể gửi trực tiếp dữ liệu cho `client 2` cả. Chỉ có một cách duy nhất đó là thông qua 1 server trung gian ở giữa đóng vai trò "trung chuyển" dữ liệu như hình mô tả dưới đây

![File_000](https://user-images.githubusercontent.com/15076665/191404991-c978e598-782e-4f00-9048-f303aad650bb.png)

OK, vậy là bài toán đã được xác định, lúc này cách giải quyết là như thế nào ?

### Cách giải quyết

Với tương tác từ `client 1` đến `server` ta có thể sử dụng lời gọi API thông thường được, nhưng tương tác từ `server` đến `client 2` thì **KHÔNG THỂ** sử dụng "lời gọi API được". Theo lí thuyết ta có thể sử dụng một trong 3 phương thức dưới đây:

- Polling
- Long Polling
- websocket

Với `Polling` thì như hình bên dưới client sẽ "liên tục hỏi" server về việc có dữ liệu mới hay không ? Tần suất hỏi này được thiết lập dựa theo "tần suất polling". Trong trường hợp câu trả lời là **không** liên tục thì việc "hỏi liên tục" này sẽ gây **lãng phí thời gian** và **lãng phí tài nguyên**

![Screen Shot 2022-09-15 at 23 26 56](https://user-images.githubusercontent.com/15076665/190430305-af1c63c2-4105-47d2-871e-6a34f8ba594b.png)

Do `Polling` hoạt động "không hiệu quả" nên ta có một cách tiếp cận khác là `Long Polling`, như hình mô tả bên dưới client sẽ "giữ" cho connection tồn tại cho đến khi:

- Nhận được data trả về từ server
- Đạt tới ngưỡng `timeout`

Khi client nhận được data trả về từ server thì nó sẽ gửi request đến server để tiếp tục quá trình `Long Polling` như trên. Tuy vậy `Long Polling` vẫn có những nhược điểm như sau:

- `client 1` và `client 2` ở trong bài toán phía trên "có thể" sẽ không kết nối tới cùng một server (đây cũng là một vấn đề mà tôi gặp phải trong quá trình dev, vấn đề này sẽ được trình bày ở phần kế). Nguyên nhân do HTTP là `stateless` - tức là server sẽ không biết gì về trạng thái của client, nên nếu `load balancer` sử dụng giải thuật `round robin` để phân bổ request thì "có thể" server nhận được data không kết nối được với client cần nhận data
- Server cũng không thể báo cho client biết được khi kết nối bị ngắt
- Trong trường hợp data không được gửi quá nhiều thì `request long polling` vẫn được gửi liên tục khi `timeout`

![Screen Shot 2022-09-16 at 8 14 56](https://user-images.githubusercontent.com/15076665/190525180-b8ba06b5-28e3-40f6-a230-58e2b129de6c.png)

Cách tiếp cận cuối cùng ở đây là `websocket`, đây là một cách tiếp cận phổ biến giúp server có thể cập nhật các sự thay đổi cho client.

![Screen Shot 2022-09-17 at 10 46 43](https://user-images.githubusercontent.com/15076665/190835781-49d79114-6b27-4df5-8d27-4b0b74d945f2.png)

Nói qua thì websocket là một kết nối hai chiều (bi-directional) được khởi tạo ở phía client. Khi khởi tạo thì nó vẫn là `HTTP connection` nhưng sẽ được "upgrade" để trở thành `websocket connection` thông qua các bước "handshake". Ở đây tôi không tiện nói sâu về quá trình này nhưng bạn có thể hiểu nôm na như sau:

B1: Một kết nối HTTP/1.1 vẫn sẽ được tạo ra giữa client và server

B2: Client sẽ gửi một "upgrade" HTTP request với `header` bao gồm tối thiểu 2 thông tin sau

```JSON
Connection: Upgrade
Upgrade: websocket
```

B3: Client chờ đến khi server reply lại response có chứa `HTTP 101 Switching Protocols`, response này cho thấy server đang chuyển sang protocol mà client yêu cầu upgrade trong header của request gửi lên

B4: Sau khi nhận được response thì websocket connection sẽ được thiết lập

Chi tiết hơn về websocket các bạn có thể tìm đọc ở 2 link bên dưới:

- <https://sookocheff.com/post/networking/how-do-websockets-work/>
- <https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism>

Trở lại với vấn đề của chúng ta, với websocket connection "bền vững" này server hoàn toàn có thể cập nhật mọi thay đổi cho phía client đồng thời việc quản lí kết nối cũng sẽ nằm hoàn toàn ở phía server

*Chú thích*: những phần về `Polling`, `Long Polling`, `websocket` tôi đều tham khảo từ sách `System Design Interview - P1` của tác giả `Alex Xu`. Các bạn có thể tìm và mua tại [link](https://www.amazon.co.jp/System-Design-Interview-insiders-Second/dp/B08CMF2CQF/ref=asc_df_B08CMF2CQF/?tag=jpgo-22&linkCode=df0&hvadid=453560710311&hvpos=&hvnetw=g&hvrand=2259595574916540920&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=1009309&hvtargid=pla-934212337151&psc=1&th=1&psc=1)

## Giải pháp

Với bài toán của mình tôi quyết định sử dụng `socket.io`. Nói qua cho những bạn chưa biết thì `socket.io` là một thư viện khá nổi tiếng để triển khai `websocket connection` cho các dự án `Realtime chat`, ...

Trang chủ của nó là: <https://socket.io/>

Dự án tôi đảm nhận có sử dụng `nestjs` (<https://nestjs.com/>) cho server và `ReactJS` (<https://reactjs.org/>) cho phía web (client 2). Để tiện cho việc minh hoạ bài toán, tôi sẽ sử dụng `ReactJS` cho cả `client 1` và `client 2`

Các bạn có thể tham khảo code minh hoạ ở repository sau: <https://github.com/tuananhhedspibk/BlogCode/tree/main/SocketIoEmitter>

Về cơ bản hai client 1 & 2 sẽ tương tác qua lại như hình minh hoạ bên dưới, `client 1` sẽ gửi dữ liệu (ở đây là message) cho `client 2`.

![Screen Shot 2022-09-21 at 18 34 34](https://user-images.githubusercontent.com/15076665/191470565-5058b2e2-db06-4941-b00b-513bcb18ac5e.png)

### Giải thích cụ thể

**client 1:**

<https://github.com/tuananhhedspibk/BlogCode/tree/main/SocketIoEmitter/client1/src/App.tsx#L11>

```TS
// client 1 sẽ gửi message đến server thông qua lời gọi API POST
// Ở đây server chạy ở cổng 6969 của localhost

const submitFormHandler = async (e: any) => {
  e.preventDefault();

  return axios.post('http://localhost:6969/app/push-data', {
    message,
  });
};
```

Ở client 1 tôi sử dụng thư viện `axios` để gửi API POST request lên server. Thông tin về thư viện `axios` bạn có thể xem ở link <https://github.com/axios/axios>

**Server:**

<https://github.com/tuananhhedspibk/BlogCode/tree/main/SocketIoEmitter/server/src/app.controller.ts#L10>

```TS
// Server sẽ nhận dữ liệu gửi lên từ client
// Sau đó sẽ truyền sang xử lí phía socket

@Post('/push-data')
pushData(@Body() body: any): void {
  console.log(`Received message: ${body.message} from ${req.headers.origin} by http connection`);
  return this.eventGateway.sendData(body.message);
}
```

<https://github.com/tuananhhedspibk/BlogCode/tree/main/SocketIoEmitter/server/src/events/events.gateway.ts#L13>

```TS
// Server socket sẽ emit một event có kèm dữ liệu là message
// Client nào có kết nối websocket tới server và lắng nghe event "EmitData" sẽ nhận được message

export class EventsGateway {
  @websocketServer()
  server: Server;

  sendData(message: string) {
    this.server.emit('EmitData', message);
  }
}
```

Cách triển khai socket với nestjs bạn có thể tham khảo ở 2 link bên dưới:

- <https://docs.nestjs.com/websockets/gateways>
- <https://github.com/nestjs/nest/tree/master/sample/02-gateways>

**client 2:**

<https://github.com/tuananhhedspibk/BlogCode/tree/main/SocketIoEmitter/client2/src/App.tsx#L5>

```TS
// client 2 sẽ tạo kết nối websocket tới Server

import { io } from 'socket.io-client';

const socket = io('http://localhost:6969');
```

<https://github.com/tuananhhedspibk/BlogCode/tree/main/SocketIoEmitter/client2/src/App.tsx#L10>

```TS
// Socket ở client 2 sẽ "lắng nghe" event "EmitData" được phát sinh từ phía server
// Do lắng nghe sự kiện này nên client 2 sẽ lấy về được message được truyền đi từ server

socket.on('EmitData', (payload) => {
  setMessage(payload);
});
```

Ở phía client 2 tôi sử dụng `socket.io-client` để triển khai, bạn có thể tham khảo chi tiết về việc cài đặt cũng như sử dụng thư viện này ở link <https://socket.io/docs/v4/client-initialization/>

BINGO !!! Vậy là bài toán của chúng ta đã được giải quyết.

Nhưng đời không như mơ :) với môi trường `local` hay `staging` thì cách triển khai này hoạt động bình thường nhưng với `production` thì **KHÔNG**, đúng vậy **KHÔNG HOẠT ĐỘNG** :)

Vấn đề ở đây là hệ thống không hề đưa ra bất kì ngoại lệ (exception) hoặc lỗi nào cả, đó mới là điều khiến tôi đau đầu.

## Mô phỏng lại "hiện trường"

Ta thấy rằng `client 1` không thể gửi message sang cho `client 2` dù hệ thống không báo lỗi (WTF ???).
Sau đây tôi sẽ tái hiện lại "hiện trường" để các bạn tiện hình dung.

### Server & Client

Ở phía server tôi chỉ có một sự thay đổi ở phần thiết lập cho socket gateway, cụ thể là tôi chỉ định phương thức kết nối là `websocket` thay vì sử dụng phương thức mặc định là `polling`.

Thực chất thì giá trị mặc định ở đây là `['polling', 'websocket']` tức là nestjs sẽ sử dụng `polling` cho socket gateway, trong trường hợp `polling` thất bại thì nestjs sẽ sử dụng `websocket` để thay thế.

```TS
@websocketGateway({
  cors: {
    origin: '*',
  },
  transports: ['websocket'],
})
```

Cụ thể hơn các bạn có thể xem ở file: <https://github.com/tuananhhedspibk/BlogCode/blob/main/SocketIoEmitter/server/src/events/events.gateway.ts#L7>

Ở phía `client 2` (client lắng nghe event từ server) tôi cũng thêm tuỳ chọn `transports` vào constructor `io()` của `socket.io-client`

```TS
const socket = io('http://localhost:7000', { transports: ['websocket'] });
```

Cụ thể hơn các bạn có thể xem ở file: <https://github.com/tuananhhedspibk/BlogCode/blob/main/SocketIoEmitter/client2/src/App.tsx#L5>

Giờ tôi sẽ tạo ra 2 server instances lần lượt lắng nghe ở 2 cổng `6969` và `7000` bằng cách truyền tham số dòng lệnh `PORT` vào câu lệnh khởi tạo server

```shell
PORT=6969 yarn start:dev # server 1
PORT=7000 yarn start:dev # server 2
```

OK, vậy là 2 server instances đều đã được bật. Vấn đề tiếp theo là làm cách nào để 2 clients lần lượt connect đến các server khác nhau đôi một ở đây. Cách đơn giản nhất là chỉ định `port` của server ở từng client là khác nhau.

- Với `client 1` tôi sẽ cho nó kết nối tới cổng `6969` - `server 1`
- Với `client 2` tôi sẽ cho nó kết nối tới cổng `7000` - `server 2`

### Kết quả

"Hiện trường" của chúng ta trông sẽ như thế này

![File_000](https://user-images.githubusercontent.com/15076665/192092261-d9e455ed-f239-49d7-bd28-f9bf7d877f4f.png)

Như hình trên hẳn bạn cũng có thể thấy rằng hai client 1 và 2 của chúng ta sẽ không kết nối tới cùng 1 server dẫn đến tình trạng message gửi từ client 1 không thể tới được client 2.

Chúng ta cũng có thể thấy trực tiếp trên demo code như hình bên dưới

![Screen Shot 2022-09-24 at 12 43 28](https://user-images.githubusercontent.com/15076665/192078743-9411279d-fed5-4b6d-bfd3-eb4868ef8227.png)

Cụ thể là `client 2` (chạy ở cổng `3002`) kết nối socket tới `server 2`, còn `client 1` (chạy ở cổng `3001`) lại kết nối tới `server 1` nên message từ `client 1` sẽ được gửi đến `server 1` thay vì `server 2` dẫn đến tình trạng **client 2 KHÔNG NHẬN ĐƯỢC MESSAGE TỪ client 1**

Vấn đề là như vậy còn giải pháp sẽ như thế nào đây ???

## Giải pháp cho vấn đề phát sinh

Nghĩ một cách đơn giản chắc hẳn bạn đọc cũng có thể nảy ra một ý kiến đó là kết nối 2 servers này lại để chúng có thể truyền dữ liệu sang nhau và cuối cùng là truyền tới client thứ 2 kia.

Chính xác, giải pháp của chúng ta ở đây là vậy, nhưng triển khai như thế nào ???

Rất may là `socket.io` cũng đã "nghĩ" đến tình huống này và phát triển cho chúng ta một thư viện tuyệt với đó là `@socket.io/redis-adapter` (link github: <https://github.com/socketio/socket.io-redis-adapter>). Như trang chủ của socket.io (<https://socket.io/docs/v4/adapter/>) có nói rằng adapter là một component phía server có chức năng "broadcast" các events tới mọi clients và khi triển khai socket cho nhiều servers chúng ta cần phải thay thế `in memory adapter default` bằng các implementation khác và ở đây tôi chọn `Redis Adapter`

Và thật là kì diệu `nestjs` cũng hỗ trợ cho phép chúng ta có thể triển khai `redis adapter` một cách dễ dàng (bạn có thể tham khảo tại <https://docs.nestjs.com/websockets/adapter>)

Các bước triển khai `redis adapter` được tôi tiến hành như sau:

B1: Dựng redis instance (triển khai redis adapter mà không cần redis mới là lạ :) )

B2: Triển khai redis adapter

### Dựng redis instance

Các bạn có thể tham khảo cách cài đặt `redis-cli` ở link này <https://redis.io/docs/getting-started/installation/>

Sau khi khởi động thì redis instance sẽ lắng nghe ở cổng `6379`

### Triển khai redis adapter

Các bạn có thể làm theo hướng dẫn của nestjs tại <https://docs.nestjs.com/websockets/adapter>

Còn đây là phần triển khai của tôi

- Redis IO Adapter: <https://github.com/tuananhhedspibk/BlogCode/blob/main/SocketIoEmitter/server/src/events/redis.adapter.ts>
- Attach to main server app: <https://github.com/tuananhhedspibk/BlogCode/blob/main/SocketIoEmitter/server/src/main.ts#L17>

Và cuối cùng thì

![Screen Shot 2022-09-24 at 16 56 03](https://user-images.githubusercontent.com/15076665/192087270-c7c5ec28-e298-4d12-be82-0723678435dc.png)

![Screen Shot 2022-09-24 at 16 56 11](https://user-images.githubusercontent.com/15076665/192087271-f86a3f89-0b74-49fd-a9b4-8d67100eb771.png)

bạn thấy đó, message từ client 1 đã được gửi tới client 2, cụ thể là `websocket connection` ở phía `client 2` đã nhận được event `EmitData` với dữ liệu tương ứng với message được gửi đi từ `client 1`

## Cơ chế hoạt động của @socket.io/redis-adapter

Thư viện này hoạt động dựa theo cơ chế `publisher/subscriber` của Redis

Cơ chế `publisher/subscriber` có thể giải thích đơn giản như hình vẽ sau:

![File_000 (1)](https://user-images.githubusercontent.com/15076665/192089390-17e96716-aae7-4265-a223-e7a384790b90.png)

Các publishers (senders) sẽ gửi đi các messages đến channel mà không cần quan tâm đến việc receiver là ai. Phía subscriber sẽ "quan tâm" tới một channel nào đó, mỗi khi channel này có message mới thì subscriber sẽ nhận được message mà không cần biết sender (publisher) của nó là ai.

Hình dưới đây là cách hoạt động của `@socket.io/redis-adapter`

![redis io adapter](https://socket.io/images/broadcasting-redis.png)

① Khi server nhận được message, nó sẽ broadcast message tới toàn bộ clients đang kết nối tới mình.

② Các messages từ một server sẽ được publish đến `redis channel` để từ đó các servers socket.io còn lại sẽ nhận được mesages này. Đó là lí do tại sao mỗi một socket.io server vừa là `publisher` mà cũng vừa là `subscriber` như chúng ta có thể thấy ở đoạn code dưới đây:

```TS
async connectToRedis(): Promise<void> {
  const pubClient = createClient({ url: `redis://127.0.0.1:6379` }); // use to publish message
  const subClient = pubClient.duplicate(); // use to receive message

  await Promise.all([pubClient.connect(), subClient.connect()]);

  this.adapterConstructor = createAdapter(pubClient, subClient);
}
```

Do đó khi `client 1` gửi request chứa message lên `server 1`, server này sẽ publish message đó sang redis channel, `server 2` subscribe redis channel sẽ nhận được message để từ đó "broadcast" đến cho `client 2` đang kết nối tới nó. Và kết quả là phía màn hình của client 2 có thể hiển thị được message được gửi từ client 1.

## Kết

Trên đây là một vài chia sẻ của tôi về `nestjs`, `socket.io` và `@socket.io/redis-adapter` hi vọng các bạn có thể dùng nó làm tư liệu tham khảo cho những project cá nhân cũng như trong công việc của mình. Cảm ơn các bạn đã kiên nhẫn đọc hết bài viết, hẹn gặp lại ở một bài viết khác.
