# How To Do Code Reviews Properly

Link: https://medium.com/@techworldwithmilan/how-to-do-code-reviews-properly-9fa59d932990

Những lợi ích của code review:

- Đảm bảo sự thống nhất trong thiết kế và triển khai.
- Tối ưu hoá code nhằm đem lại hiệu năng tốt hơn.
- Cơ hội để học hỏi
- Chia sẻ kiến thức, mentoring
- Tăng cường sự gắn kết trong team

## Cần check những gì khi tiến hành code review

Hãy kiểm tra những điều sau khi tiến hành code review:

1. **Chức năng và thiết kế**: liệu rằng sự thay đổi lần này có theo đúng như kiến trúc sẵn có hoặc có tích hợp tốt với phần còn lại của hệ thống hay không? Liệu nó có tính gắn kết cao và tính phụ thuộc thấp giữa các components hay không? Nó có tuân thủ các quy tắc như OO, SOLID, DRY, KISS, YAGNI hay không?
2. **Triển khai**: chúng ta kiểm tra xem liệu rằng logic triển khai có đúng hay không? Nếu code phức tạp hơn mức bình thường thì liệu chúng có cần thiết hay không? Chúng ta cũng kiểm tra xem liệu chúng ta có vận dụng tốt các design pattern hay không.
3. **Testing**: ở phần này chúng ta sẽ kiểm tra xem tất cả các tests có pass hay không, liệu chúng ta có sử dụng unit tests cho mọi code paths hay behaviors hay không, chúng ta có sử dụng integration test cho external system như DBs, ... Liệu rằng các trường hợp biên có được check hay không và code có phủ được 60% - 80% hay không?
4. **Documentation**: liệu rằng PR description đã được thêm hay chưa? Liệu giải pháp của chúng ta đã được ghi trong README.md hay chưa?
5. **Code styles**: liệu chúng ta có đang follow sát theo coding styles hay không? Liệu rằng việc chọn tên (class name, method name) như vậy đã chính xác hay chưa?

## Một vài mẹo khi tiến hành code review

### Review code của chính mình trước

Trước khi gửi code cho người khác review, hãy tự review code của chính mình trước. Đọc code của mình mà không sử dụng text editor hay IDE đang sử dụng. Tìm những phần khiến bản thân mình cảm thấy nghi ngờ.

### Viết một đoạn mô tả ngắn về những gì đã thay đổi

Viết về những gì thay đổi ở high-level và tại sao lại có sự thay đổi đó.

### Tự động hoá những gì có thể tự động

Ví dụ như CI, linter, automated tests và static code analysis. Khi chúng ta tích hợp nó trên PR level, nó sẽ chạy trên mọi PR và block code merging.

### Hãy thực hiện những hành động kick-off có ý nghĩa với team members

Hãy tiến hành những buổi kickoff với code owner để có một cái nhìn tổng quan trước khi tiến hành review code để từ đó giảm bớt effort khi tiến hành review

### Không vội vàng

Bạn cần phải hiểu những gì đang thay đổi trong code - từng dòng. Đọc đi đọc lại nhiều lần nếu cần thiết, class by class. Có những đoạn code cần phải đọc nhiều lần nhưng cũng có những đoạn code chỉ cần đọc một lần là đủ. Hãy đưa ra quyết định của bạn trong quá trình review.

### Comment với sự đóng góp

Đừng chỉ tập trung vào sự thay đổi hoặc suggest mà hãy để lại những commment tích cực, những lời khen, ... Ngoài ra hãy đưa ra những lời khuyên để cải thiện.

### Approve PR khi cảm thấy đủ tốt

Không cần phải quá cầu toàn, khi thấy rằng PR đủ ổn, hãy approve.

### Giới hạn lượng code review

Nên giới hạn số lươnjg dòng code cho mỗi lần review. PR nên có ít files nhất có thể. Số lượng LOC lí tưởng nên là từ `200 -> 400`

### Sử dụng checklists

Checklish giúp đảm bảo việc review có thể bao quát được những tiêu chỉ cơ bản nhất.

VD:

```txt
Thank you for your contribution to this repo. Before submitting this PR, please make sure the:

- [ ] Your code builds clean without any errors or warnings
- [ ] You are using the appropriate design
- [ ] You have added unit tests and are all green
```

### Sử dụng tools

Các công cụ như BitBucket, Github sẽ hỗ trợ rất nhiều cho việc review.

## Thay đổi để review code tốt hơn 
