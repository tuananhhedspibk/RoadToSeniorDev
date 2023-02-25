# Design Log system

## Policy

- **Why**: Log nhằm mục đích gì
- **Who**: System nào tạo ra log (API, APP, ...)
- **When**: Khi nào, thời điểm nào thì đưa ra log
- **Where**: Đưa ra log ở đâu (log sẽ được gửi đến Slack, BigQuery, ...)
- **What**: Log về cái gì
- **How**: Đưa ra log như thế nào ???

## Log level

| Log level | Khái niệm | Cách đối ứng | Ví dụ |
| -------- | -------- | -------- | -------- |
| FATAL | Level này gây cản trở đến việc vận hành của hệ thống | Cần sửa ngay | Không thể kết nối tới DB |
| ERROR | Việc thực thi gây ra những lỗi ngoài dự tính | Sửa trong thời gian hoạt động của hệ thống | Không thể gửi mail |
| WARN | Không phải lỗi nhưng là những vấn đề như: input không như mong muốn hoặc thực thi không như mong muốn | Refactor định kì | API xoá dữ liệu một cách định kì |
| INFO | Thông báo khi bắt đầu hoặc kết thúc một xử lí, transaction. Cũng có thể là việc đưa ra các thông tin cần thiết khác | Không cần phải sửa | Đưa ra nội dung của req / res, hoặc khi bắt đầu hoặc kết thúc batch |
| DEBUG | Thông tin liên quan đến trạng thái hoạt động của hệ thống | Không đưa ra ở môi trường production | Có thể đặt ở các hàm bên trong app |
| TRACE | Thông tin chi tiết ở mức độ cao hơn DEBUG | Không đưa ra ở môi trường production |  |

## Case

### System log

Những log này sẽ sử dụng để debug khi hệ thống có lỗi

- Who: bản thân hệ thống
- When: Khi phát sinh lỗi
- Where:
  - FATAL / ERROR: Thông báo đến một kênh mà dev có thể nhận ra và xử lí kịp thời
  - WARN / INFO: Đưa ra bên trong hệ thống hoặc các tool quản lí log
  - DEBUG / TRACE: Đưa ra ở dạng `console.log` trên môi trường `staging`
- What:
  - FATAL / ERROR: Stacktrace
  - WARN / INFO / DEBUG/ TRACE: Nội dung muốn thông báo
- How:
  - FATAL / ERROR: Đưa ra log thông qua các tool quản lí log hoặc đưa tới Slack, SMS, ... (dưới hình thức `push`)
  - WARN / INFO / DEBUG / TRACE: Đưa ra log thông qua các tool quản lí log hoặc bên trong hệ thống (dưới hình thức `pull`)

### Access log

Chủ yếu là các log liên quan đển gửi, nhận request

- Who: Bản thân system hoặc từ phía infra
- When: Tại thời điểm access (gửi, nhận request)
- Where: Chủ yếu là log ở `level INFO`, nên sẽ đưa ra ở dạng pull. Ngoài ra do lượng log có thể nhiều nên tốc độ tìm kiếm log cũng là việc cần phải chú ý
- What: Đưa ra **ai**, **bằng cách nào**, **khi nào** access đến hệ thống
- How: Tuỳ vào đối tượng và mục tiêu mà có sự khác biệt

### Action log

Đưa ra để tiến hành phân tích, qua đó nhằm cải thiện service hoặc application

- Who: Bản thân system hoặc external tools
- When: Khi phát sinh ra action (hành động)
- Where: Các tool có khả năng phân tích log (BigQuery, BI, ...)
- What: Tuỳ vào mục tiêu
- How: Tuỳ vào đối tượng và mục tiêu mà có sự khác biệt

### Auth log

Là những log được đưa ra khi login, logout hoặc khi tiến hành authentication.

- When: Khi tiến hành authentication
- Where: Chủ yếu là `level INFO`, dưới hình thức PULL
- What: Đưa ra **ai**, **bằng cách nào**, **khi nào** thì được authenticate
- How: Tuỳ vào phương thức authenticate mà có sự khác biệt
