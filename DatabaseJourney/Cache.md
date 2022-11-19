# Cache Aside

Là cơ chế cache thường được sử dụng nhất, mô hình hoạt động của nó như sau:

![File_000](https://user-images.githubusercontent.com/15076665/202854588-a504492e-ad55-40b6-a815-ff7d6bf9b151.png)

Cơ chế này thường được dùng cho trường hợp đọc nhiều và ghi ít. Ngoài ra còn phụ thuộc vào việc dữ liệu trả về có thay đổi hay không (truy vấn theo primary key thì thường hiếm khi thay đổi).

Kĩ thuật thường dùng là `memcache`.

Lợi ích đem lại ở đây đó là khả năng phục hồi dữ liệu khi cache gặp lỗi (chỉ cần truy vấn ngược về DB là xong).

Cần lưu ý:

- Cập nhật vào cache các state create/ update của DB (sử dụng logic code)
- Cơ chế refresh cache (LRU hay LFU).

