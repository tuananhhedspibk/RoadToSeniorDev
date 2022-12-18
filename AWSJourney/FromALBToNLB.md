# Di chú từ AWS ALB sang AWS NLB

## ALB và NLB là gì ?

### Tổng quan về ALB

ALB (Application Load Balancer) hoạt động ở tầng 7 (tầng ứng dụng trong mô hình OSI)

![Screen Shot 2022-12-17 at 16 44 04](https://user-images.githubusercontent.com/15076665/208231711-ad24808d-6e27-45c9-85aa-354183a3d472.png)

ALB hỗ trợ chức năng load balancing của ứng dụng sử dụng giao thức HTTP và HTTPs

Một số tính năng chính mà ALB hỗ trợ:

- Layer-7 Load balancing: có thể tiến hành phân tải cho HTTP và HTTPs tới Target: EC2, Microservices.
- Hỗ trợ HTTPs.
- Redirect: có thể tiến hành redirect request từ URL này sang một URL khác.

Trong quá trình tạo ALB ta **cần** phải có `Security Group` cho ALB.

### Tổng quan về NLB

NLB (Network Load Balancer) hoạt động ở tầng 4 (tầng giao vận trong mô hình OSI)

![Screen Shot 2022-12-17 at 16 58 42](https://user-images.githubusercontent.com/15076665/208232092-733eedf9-997b-4162-8e17-a6bcd7dab2ec.png)

NLB hỗ trợ load balancing cho các ứng dụng TCP, UDP, TCP_UDP và cả TLS

NLB được thiết kế để đáp ứng các ứng dụng với lưu lượng truy cập cao, NLB có khả năng xử lí "cả triệu requests trên giây" với độ trễ cực thấp.

Một số tính năng chính mà NLB hỗ trợ:

- Layer-4 Load balancing: có thể tiến hành phân tải cho TCP hay UDP traffic tới các Target: EC2, Microservices.
- Độ trễ thấp.
- Giảm tải TLS: NLB hỗ trợ **client TLS session termination** (mã hoá và giải mã). Điều này cho phép ta có thể giảm tải các TLS termination tasks cho load balancer.

Trong quá trình tạo NLB ta **không cần** phải có `Security Group` cho NLB.

## Sự khác biệt giữa ALB và NLB

ALB hoạt động ở tầng-7 nên nó quan tâm tới `context`, `cookie data` nhưng NLB hoạt động ở tầng-4 nên nó chỉ quan tâm đến các thông tin về tầng mạng mà thôi.

Ngoài ra còn có một vài nhưng sự khác biệt sau:

- NLB hoạt động ở tầng giao vận và nó chỉ đơn thuần là forward request tới Target trong khi ALB có xem xét đến nội dung của HTTP request header để tiến hành routing.
- NLB chỉ quan tâm đến các thông tin ở tầng mạng và tầng giao vận nên nó không thể đảm bảo tính sẵn có của ứng dụng. Thông thường NLB sẽ "định nghĩa" tính sẵn có này thông qua khả năng server hồi đáp lại các ICMP ping hoặc server có thể đảm bảo TCP-Handshake hay không. Trong khi ALB có khả năng làm việc đó không chỉ dựa trên HTTP Get request thành công đối với một trang cụ thể mà còn có thể kiểm chứng rằng **nội dung có đúng như kì vọng dựa theo parameter hay không**
- NLB không có khả năng phân biệt giữa 2 ứng dụng A và B khi chúng cùng sử dụng một địa chỉ IP, trong khi ALB thì có thể, do đó sẽ dẫn đến trường hợp NLB có khả năng gửi request đến một ứng dụng "crashed" hoặc "offline" trong khi ALB thì không hề.

## Hệ thống hiện thời

![Screen Shot 2022-12-17 at 18 03 08](https://user-images.githubusercontent.com/15076665/208234453-d55c3722-7b02-465c-ac95-c77f9f5427e7.png)

### Mô tả hệ thống

Giải thích đơn giản thì hệ thống của tôi phục vụ cho cả **người dùng web** và **người dùng mobile** nên tôi dựng nên 2 hosts:

- www.web.com: dùng cho web
- api.com: dùng cho ứng dụng mobile gọi đến API của hệ thống

Hai hosts này của tôi cùng có `Endpoint` là một ALB duy nhất. Tại ALB này tôi sẽ tiến hành routing dựa theo host cũng như URI path như sau:

- Nếu người dùng truy cập bằng giao thức `HTTP`, tôi sẽ redirect tới giao thức `HTTPs`
- Nếu host = `api.com` tôi sẽ routing tới `api-target-group` (nối với nó là các EC2 API instances)
- Nếu host = `web.com`, tôi sẽ tiến hành check `URI path`, với các `URI path` có dạng `/api` tôi sẽ routing tới `api-target-group` (nối với nó là các EC2 API instances)
- Các trường hợp còn lại và default tôi đều cho routing tới `web-target-group` (nối với nó là các EC2 Web instances)

Bạn có thể xem hình minh hoạ dưới đây để biết thêm chi tiết:

![Screen Shot 2022-12-17 at 18 13 05](https://user-images.githubusercontent.com/15076665/208234790-da046c99-f824-41a0-8939-1cb26821b134.png)

### Vấn đề gặp phải

Hiện tại hệ thống trên gặp phải một "vấn đề" đó là vào các khung giờ cao điểm, khi số lượng người truy cập gia tăng (ở cả ứng dụng mobile cũng như web) thì tôi "buộc" phải thực hiện hai việc sau:

- Tiến hành "xin" AWS gia tăng access capacity của ALB để nó có thể chịu được nhiều tải hơn vì nếu không tăng access capacity thì sẽ dẫn đến tình trạng "bottleneck" và làm cho hệ thống bị chậm đi trông thấy.
- Tiến hành tăng cường thêm các EC2 instances để xử lí nhiều requests tới hệ thống hơn.

Hiện thời thì việc tăng cường số lượng các EC2 instances là việc buộc phải làm, không thể thoái thác được. Nhưng việc "xin" AWS gia tăng access capacity của ALB là việc có thể hạn chế (tôi muốn hạn chế là vì cần phải "nộp đơn" trước 3 ngày thì phía quản trị AWS mới đồng ý tăng dung lượng và đương nhiên là không ai thích chờ đợi cả).

Do đó để có thể "vứt bỏ" việc phải đi "xin xỏ" quản trị viên AWS gia tăng access capacity của ALB tôi quyết định sử dụng NLB vì một đặc tính rất hay của nó đó là:

> NLB được thiết kế để đáp ứng các ứng dụng với lưu lượng truy cập cao, NLB có khả năng xử lí "cả triệu requests trên giây" với độ trễ cực thấp.

## Hệ thống sau khi di chú sang NLB

Sau khi di chú sang NLB, hệ thống của tôi trông sẽ như dưới đây

![Screen Shot 2022-12-17 at 21 33 05](https://user-images.githubusercontent.com/15076665/208242059-d126c869-bbaa-43f9-a63f-0946b02d7005.png)

### Vấn đề gặp phải sau di chú

Tất nhiên sẽ có những vấn đề phát sinh đi kèm. Dưới đây là các vấn đề mà tôi gặp phải:

- Làm cách nào để phía bên web có thể gọi được sang API ??? (trước đây nhiệm vụ routing lời gọi từ web sang API là do ALB đảm nhận, nhưng như đã nói ở trên NLB sẽ **KHÔNG LÀM NHIỆM VỤ ROUTING** mà thay vào đó nó sẽ **FORWARD THẲNG REQUEST TỚI TARGET LUÔN**)
- Làm cách nào để redirect từ `HTTP` sang `HTTPs` do với NLB ta không thể thiết lập rule để redirect như với ALB được ???

### Giải quyết vấn đề

Trên thực tế, trong EC2 web instance của tôi, ngoài code của web tôi còn sử dụng thêm cả nginx như một `Reverse Proxy` chắn trước trang web của mình.

*Các bạn có thể tham khảo thêm về Reverse Proxy tại [đây](https://viblo.asia/p/forward-proxy-vs-reverse-proxy-7ymJXXQ6Jkq)*

Do đó tôi quyết định, toàn bộ việc `routing sang API` hay `redirect từ HTTP sang HTTPs` sẽ do **nginx** đảm nhận.

Với lời gọi API tôi xử lí như sau:

```nginx
server {
  listen 80 default_server;

  location /api* {
    proxy_pass https://api.com; 
  }
}
```

Lúc này quá trình routing API sẽ diễn ra như hình minh hoạ bên dưới:

![Screen Shot 2022-12-17 at 22 11 28](https://user-images.githubusercontent.com/15076665/208243543-9151df53-c940-468c-b321-f4927abf85a1.png)

Giải thích đơn giản như sau: những lời gọi API từ web sẽ có URL như sau: `https://www.web.com/api/*`, những lời gọi này đều không được NLB xử lí mà truyền thẳng tới EC2 instance luôn. Tại đây, nginx thấy rằng URI "đồng dạng" với `/api*` nên nginx sẽ `forward` lời gọi này sang `host của API` là `https://api.com`.

OK, vậy là xong phần routing cho API. Còn phần redirect từ `HTTP` sang `HTTPs` tôi sẽ làm như sau.

Tại NLB cho web, tôi thiết lập 2 listeners:

- TCP:80 - Dùng cho kết nối `HTTP`
- TLS:443 - Dùng cho kết nối `HTTPs`

Tương ứng với mỗi listener này sẽ là các TargetGroup khác nhau đôi một:

- TCP:80 - http-web-target-group
- TLS:443 - https-web-target-group

Chú ý: nếu với `ALB` ta cần thiết lập TargetGroup với protocol là `HTTP` thì với `NLB` ta cần thiết lập protocol là `TCP`

Với `http-web-target-group`, tôi sẽ cho lắng nghe ở cổng `81`

Với `https-web-target-group`, tôi sẽ cho lắng nghe ở cổng `80` - đây cũng chính là cổng mặc định mà nginx hiện tại đang lắng nghe

Do đó, để xử lí `http-web-target-group`, tôi buộc phải cho nginx lắng nghe thêm ở cổng `81` nữa, thiết lập cho cổng này sẽ rất đơn giản, chỉ đơn thuần là **LUÔN LUÔN REDIRECT SANG HTTPS** mà thôi.

```nginx
server {
  listen 81 default_server;

  return 301 https://web.com$request_uri;
}
```

Để bạn đọc tiện hình dung, tôi có vẽ lại mô hình như sau:

![Screen Shot 2022-12-17 at 22 06 28](https://user-images.githubusercontent.com/15076665/208243348-b040a0e5-7356-4ebc-84a8-91aec3e2cda7.png)

Giải thích một cách đơn giản thì khi người dùng truy cập vào địa chỉ `http://www.web.com`, request sẽ được NLB chuyển tới `http-web-target-group`, target group này sẽ chuyển đến EC2 instance, tại đây nginx đang lắng nghe ở cổng 81 sẽ nhận được request và redirect nó sang địa chỉ `https://www.web.com`

Đó là về phần routing và redirect. Đi sâu một chút nữa xuống mức `Security Group` ta sẽ thấy rằng, mọi request đều được NLB forward thẳng đến EC2 instance do đó với các EC2 instance ta cũng cần "mở toang" 2 cổng `80` và `81` cho mọi địa chỉ IP - cụ thể là `0.0.0.0/0` với giao thức `HTTP` để EC2 instance có thể nhận được mọi requests đến từ các người dùng khác nhau (tôi áp dụng Security Group dạng này cho cả `EC2 web instance` và `EC2 api instance`).

## Hình hài cuối cùng của hệ thống

Sau quá trình thiết lập đầy "gian khổ" như trên, hệ thống của tôi trông sẽ như sau:

![Screen Shot 2022-12-17 at 22 25 45](https://user-images.githubusercontent.com/15076665/208244147-09777d8c-873c-4d55-aa6c-596c2dfb58c2.png)

Trông có vẻ khá phức tạp so với hệ thống cũ nhưng lúc này NLB sẽ không còn "vất vả" như ALB nữa khi vừa đảm nhận nhiệm vụ forward request vừa phải đảm nhận nhiệm routing, nên sau khi release xong tôi không cần phải "xin xỏ" AWS để gia tăng dung lượng xử lí cho Load balancer của mình nữa.

Chắc chắn sau này sẽ còn nhiều vấn đề phát sinh hơn nữa khi sử dụng NLB, hi vọng rằng lúc đó tôi vẫn có thể xử lí và tiếp tục chia sẻ những kinh nghiệm của cá nhân cho bạn đọc.

Cảm ơn các bạn đã đọc hết. Hẹn gặp lại ở các bài viết tiếp theo.

## Tài liệu tham khảo

- https://medium.com/awesome-cloud/aws-application-load-balancer-alb-overview-introduction-to-amazon-alb-what-is-aws-alb-b5280f625153
- https://medium.com/awesome-cloud/aws-network-load-balancer-nlb-overview-introduction-to-amazon-nlb-what-is-aws-nlb-elb-837749c20063
- https://medium.com/awesome-cloud/aws-difference-between-application-load-balancer-and-network-load-balancer-cb8b6cd296a4
