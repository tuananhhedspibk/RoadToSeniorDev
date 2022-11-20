# Cache Aside

## Overview

Là cơ chế cache thường được sử dụng nhất, mô hình hoạt động của nó như sau:

![File_000](https://user-images.githubusercontent.com/15076665/202854588-a504492e-ad55-40b6-a815-ff7d6bf9b151.png)

Cơ chế này thường được dùng cho trường hợp đọc nhiều và ghi ít. Ngoài ra còn phụ thuộc vào việc dữ liệu trả về có thay đổi hay không (truy vấn theo primary key thì thường hiếm khi thay đổi).

Kĩ thuật thường dùng là `memcache`.

Lợi ích đem lại ở đây đó là khả năng phục hồi dữ liệu khi cache gặp lỗi (chỉ cần truy vấn ngược về DB là xong).

Cần lưu ý:

- Cập nhật vào cache các state create/ update của DB (sử dụng logic code)
- Cơ chế refresh cache (LRU hay LFU).

## Least Frequently Used (LFU)

### Nội dung giải thuật

Là giải thuật caching, tại đó cache block với số lần được truy xuất ít nhất sẽ bị loại bỏ nếu cache bị đầy. Ở giải thuật này ta sẽ kiểm tra tần suất được truy xuất của từng block, các block cũ với tần suất bằng nhau sẽ bị loại bỏ dựa theo một quy tắc nào đó (ví dụ FIFO chẳng hạn, hoặc sẽ loại bỏ đi block với lần sử dụng gần nhất cách thời điểm hiện tại xa nhất).

### Cách hoạt động

Sử dụng 2 hàm `add` & `get`.

- `add(key, value)`: thêm cặp `(key, value)` tương ứng vào cache, đầu tiên kiểm tra capacity, nếu còn dư thì sẽ thêm còn nếu không thì sẽ loại bỏ đi `(key, value)` với tần suất sử dụng thấp nhất và thời gian sử dụng cuối xa nhất.
- `get(key)`: trả về value tương ứng với key (nếu key không tồn tại, ta trả về giá trị integer nhỏ nhất) đồng thời di chuyển nó đến vị trí thích hợp trong cache (dựa theo tần số).

![File_000 (1)](https://user-images.githubusercontent.com/15076665/202880934-05dc5e1a-4e69-465a-9549-6980765ea0da.png)

### Code implementation

<https://github.com/tuananhhedspibk/BlogCode/tree/main/LFUCaching>

## Least Recently Used (LRU)

### Nội dung chính

Giải thuật này giữ cho các items thường xuyên được sử dụng luôn có mặt ở phần đầu của cache. Khi một item mới được truy cập thì nó sẽ được đẩy lên đầu. Khi cache chạm tới ngưỡng capacity các items ít được truy cập sẽ bị xoá khỏi cache tính từ phía cuối của cache

### Hoạt động

LRU cache cung cấp 2 method `put` & `get`

- `put(key, value)`: set hoặc insert cặp `(key, value)`. Khi cache chạm tới ngưỡng capacity, item với tần suất truy cập ít nhất sẽ bị loại bỏ khỏi cache (sẽ là phần tử cuối cùng).
- `get(key)`: lấy về value tương ứng với key, nếu key không tồn tại sẽ trả về -1, nếu key tồn tại thì sẽ đưa item tương ứng với key lên đầu cache.

Ví dụ:

![LRU caching](https://user-images.githubusercontent.com/15076665/202895375-cf4d19e4-5359-45b1-8c8b-b431bfffc429.png)

### Triển khai thuật toán

<https://github.com/tuananhhedspibk/BlogCode/blob/main/LRUCaching/index.ts>
