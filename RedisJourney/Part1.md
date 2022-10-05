# Redis basic - Part 1

## SET command

`SET key value XX` - chỉ chạy nếu key đã tồn tại

`SET key value NX` - chỉ chạy nếu key chưa từng tồn tại

Các câu lệnh `SET` nếu thành công sẽ trả về `OK`, nếu không được thực thi sẽ trả về `nil` hoặc `NULL`

`SET key value EX | PX | EXAT` - dữ liệu sẽ expire sau một khoảng thời gian nhất định

VD:

`SET color red EX 2` - color sẽ bị huỷ sau 2 giây

Do redis "thường" được sử dụng làm cache nên nếu sau một khoảng thời gian nhất định nếu dữ liệu trong redis không được truy cập hoặc cập nhật thì dữ liệu đó nên bị xoá đi (vì redis lưu dữ liệu trong memory nên nếu redis bị "đầy" thì có thể sẽ phát sinh lỗi `out of memory`)

`SETEX key second value` - tương tự như `SET key value EX second`

`SETNX key value` - tương tự như `SET key value NX`

## MSET command

Dùng để set values cho multiple keys

`MSET key[1] value[1] key[2] value[2] ... key[n] value[n]`

`MSETNX key[1] value[1] key[2] value[2] ... key[n] value[n]` - nếu có bất kì key nào đã tồn tại thì sẽ không có lệnh `set` nào được thực hiện

## GET & MGET command

`GET key`, `MGET key[1] key[2] ... key[n]`

## DEL command

`DEL key`

## GETRANGE command

`GETRANGE key first_idx last_idx` - trả về toàn bộ kí tự từ "first_idx" đến "last_idx" của value tương ứng với key

## SETRANGE command

`SETRANGE key idx str` - thay thế substring từ idx đến str.length của value tương ứng với key