# Thiết kế hệ thống logging

Với những bạn đã và đang lập trình thì việc "check log" trong quá trình debug không còn là một điều xa lạ. Nếu không có log, chúng ta không thể biết được file nào, dòng nào trong file đó đang phát sinh lỗi để có thể sửa chữa kịp thời.

Đó là đối với những chương trình hoặc file đơn lẻ, thế nhưng với những hệ thống phục vụ người dùng trong thực tế thì việc "check log" lại càng trở nên quan trọng hơn vì nó giúp chúng ta:

- Điều tra, phát hiện ra những lỗi đang xảy ra làm ảnh hưởng đến người dùng cuối.
- Theo dõi "sức khoẻ" của hệ thống để từ đó có thể phát hiện ra những "dấu hiệu bất thường" trước khi nó kịp phát tác làm ảnh hưởng đến hệ thống.
- ...

Qua đó chúng ta có thể thấy việc "check log" đóng vai trò rất quan trọng trong quá trình phát triển cũng như vận hành một phần mềm hoặc một hệ thống (còn gọi là monitoring), thế nên việc thiết kế và triển khai một hệ thống log tốt sẽ giúp quá trình monitoring trở nên dễ dàng hơn rất nhiều.

Trong bài viết lần này tôi xin phép được gửi đến bạn đọc những kiến thức thực tế mà tôi đã tiếp thu được về việc xây dựng một hệ thống logging, một phần là để bạn đọc cảm nhận rõ hơn về tầm quan trọng của log khi vận hành hệ thống, một phần là để bạn đọc có được cho mình một tài liệu tham khảo khi muốn xây dựng một hệ thống logging trong thực tế. Rất mong được bạn đọc đón nhận.

## Logging Policy

Dưới đây là những câu hỏi chúng ta cần đặt ra trước khi xây dựng một hệ thống log.

- **Why**: Log nhằm mục đích gì ?
- **Who**: Hệ thống hay module nào tạo ra log (API, APP, ...) ?
- **When**: Khi nào, thời điểm nào thì đưa ra log ?
- **Where**: Đưa ra log ở đâu (log sẽ được gửi đến Slack, BigQuery, ...) ?
- **What**: Log về cái gì ?
- **How**: Cách thức đưa ra log là như thế nào ?

## Log level

Sau khi đã hiểu rõ được mục đích của mình khi đưa ra log, chúng ta cần "phân cấp" log.

| Log level | Khái niệm                                                                                                           | Cách đối ứng                               | Ví dụ                                                               |
| --------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------- |
| FATAL     | Level này gây cản trở đến việc vận hành của hệ thống                                                                | Cần sửa ngay                               | Không thể kết nối tới DB                                            |
| ERROR     | Việc thực thi gây ra những lỗi ngoài dự tính                                                                        | Sửa trong thời gian hoạt động của hệ thống | Không thể gửi mail                                                  |
| WARN      | Không phải lỗi nhưng là những vấn đề như: input không như mong muốn hoặc thực thi không như mong muốn               | Refactor định kì                           | API xoá dữ liệu một cách định kì                                    |
| INFO      | Thông báo khi bắt đầu hoặc kết thúc một xử lí, transaction. Cũng có thể là việc đưa ra các thông tin cần thiết khác | Không cần phải sửa                         | Đưa ra nội dung của req / res, hoặc khi bắt đầu hoặc kết thúc batch |
| DEBUG     | Thông tin liên quan đến trạng thái hoạt động của hệ thống                                                           | Không đưa ra ở môi trường production       | Có thể đặt ở các hàm bên trong app                                  |
| TRACE     | Thông tin chi tiết ở mức độ cao hơn DEBUG                                                                           | Không đưa ra ở môi trường production       |                                                                     |

## Case

Sau khi đã rõ các cấp độ log, chúng ta cần làm rõ "các loại log" mà mình muốn đưa ra. Trong phần này đối với từng loại log tôi sẽ đi trả lời 6 câu hỏi:

- Why
- Who
- When
- Where
- What
- How

như đã nói chi tiết ở phần **Logging Policy** phía trên.

### System log

- Why: Những log này sẽ sử dụng để debug khi hệ thống có lỗi.
- Who: Bản thân hệ thống sẽ sinh ra log.
- When: Đưa ra log khi phát sinh lỗi.
- Where:
  - `FATAL / ERROR`: Thông báo đến một kênh mà dev có thể nhận ra và xử lí kịp thời.
  - `WARN / INFO`: Đưa ra bên trong hệ thống hoặc các tool quản lí log.
  - `DEBUG / TRACE`: Đưa ra ở dạng `console.log` trên môi trường `staging`.
- What:
  - `FATAL / ERROR`: Stacktrace.
  - `WARN / INFO / DEBUG/ TRACE`: Nội dung muốn thông báo.
- How:
  - `FATAL / ERROR`: Đưa ra log thông qua các tool quản lí log hoặc đưa tới Slack, SMS, ... (dưới hình thức `push`).
  - `WARN / INFO / DEBUG / TRACE`: Đưa ra log thông qua các tools quản lí log hoặc bên trong hệ thống, ... (dưới hình thức `pull`).

### Access log

- Why: Đưa ra log để theo dõi việc gửi, nhận request.
- Who: Bản thân hệ thống hoặc từ phía infrastructure.
- When: Đưa ra tại thời điểm access (gửi, nhận request)
- Where: Chủ yếu là log ở `level INFO`, nên sẽ đưa ra ở dạng pull. Ngoài ra do lượng log có thể nhiều nên tốc độ tìm kiếm log cũng là việc cần phải chú ý.
- What: Đưa ra **ai**, **bằng cách nào**, **khi nào** access đến hệ thống.
- How: Tuỳ vào đối tượng và mục tiêu mà có sự khác biệt.

### Action log

- Why: Đưa ra để tiến hành phân tích hành vi của người dùng, qua đó nhằm cải thiện service.
- Who: Bản thân hệ thống hoặc external tools.
- When: Khi phát sinh ra actions (hành động).
- Where: Các tool có khả năng phân tích log (BigQuery, BI, ...).
- What: Tuỳ vào mục tiêu.
- How: Tuỳ vào đối tượng và mục tiêu mà có sự khác biệt.

### Auth log

- Why: Đưa ra nhằm mục đích theo dõi quá trình xác thực người dùng.
- Who: Bản thân hệ thống.
- When: Khi tiến hành xác thực người dùng.
- Where: Chủ yếu là `level INFO`, dưới hình thức `pull`.
- What: Đưa ra **ai**, **bằng cách nào**, **khi nào** thì được xác thực.
- How: Tuỳ vào phương thức xác thực mà có sự khác biệt

## Ví dụ minh hoạ
