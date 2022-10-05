# Redis basic - Part 1

## Set command

`SET key value XX` - chỉ chạy nếu key đã tồn tại

`SET key value NX` - chỉ chạy nếu key chưa từng tồn tại

Các câu lệnh `SET` nếu thành công sẽ trả về `OK`, nếu không được thực thi sẽ trả về `nil` hoặc `NULL`

`SET key value EX | PX | EXAT` - dữ liệu sẽ expire sau một khoảng thời gian nhất định

VD:

`SET color red EX 2` - color sẽ bị huỷ sau 2 giây

Do redis "thường" được sử dụng làm cache nên nếu sau một khoảng thời gian nhất định nếu dữ liệu trong redis không được truy cập hoặc cập nhật thì dữ liệu đó nên bị xoá đi (vì redis lưu dữ liệu trong memory nên nếu redis bị "đầy" thì có thể sẽ phát sinh lỗi `out of memory`)
