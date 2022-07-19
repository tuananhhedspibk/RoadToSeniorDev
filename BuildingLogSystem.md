# Design Log system

## Policy

- **Why: Log nhằm mục đích gì**
- Who: System nào tạo ra log (API, APP, ...)
- When: Khi nào, thời điểm nào thì đưa ra log
- Where: Ở đâu (ở trong xử lí nào, API nào, ...)
- What: Log về cái gì
- How: Như thế nào ???

## Log level

| Log level | Khái niệm | Cách đối ứng | Ví dụ |
| -------- | -------- | -------- | -------- |
| FATAL | Level này gây cản trở đến việc vận hành của hệ thống | Cần sửa ngay | Không thể kết nối tới DB |
| ERROR | Việc thực thi gây ra những lỗi ngoài dự tính | Sửa trong thời gian hoạt động của hệ thống | Không thể gửi mail |
| WARN | Không phải lỗi nhưng là những vấn đề như: input không như mong muốn hoặc thực thi không như mong muốn | Refactor định kì | API xoá dữ liệu một cách định kì |
| INFO | Thông báo khi bắt đầu hoặc kết thúc một xử lí, transaction. Cũng có thể là việc đưa ra các thông tin cần thiết khác | Không cần phải sửa | Đưa ra nội dung của req / res, hoặc khi bắt đầu hoặc kết thúc batch |
| DEBUG | Thông tin liên quan đến trạng thái hoạt động của hệ thống | Không đưa ra ở môi trường production | Có thể đặt ở các hàm bên trong app |
| TRACE | Thông tin chi tiết ở mức độ cao hơn DEBUG | Không đưa ra ở môi trường production |  |
