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
2. **Triển khai**: Chúng ta kiểm tra xem liệu rằng logic triển khai có đúng hay không? Nếu code phức tạp hơn mức bình thường thì liệu chúng có cần thiết hay không? Chúng ta cũng kiểm tra xem liệu chúng ta có vận dụng tốt các design pattern hay không.
3. **Testing**
