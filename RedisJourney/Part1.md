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

## Lợi ích của việc sử dụng các commands GETRANGE, SETRANGE, MSET, MGET

![File_000](https://user-images.githubusercontent.com/15076665/196974640-046b99ea-6b03-41ab-8134-9735117c78c8.png)

## Xử lí number

Sử dụng GET, SET, MGET, MSET, ... tương tự như với string

Với number ta còn có các lệnh như `INCR`, `DECR`, ... (các lệnh này lần lượt cộng, trừ giá trị đi 1).

> Các câu lệnh cập nhật dữ liệu trong redis sẽ diễn ra gần như lập tức, nghĩa là quá trình lấy dữ liệu ra, cập nhật và lưu dữ liệu sẽ diễn ra gần như ngay lập tức

## Ví dụ về vấn đề với việc cập nhật dữ liệu trong redis

![File_000](https://user-images.githubusercontent.com/15076665/197317434-3f21ca2f-c805-4fa3-bdf0-0a018b7e208b.png)

Lợi ích của việc sử dụng các commands như `INCR` hay `DECR` ở đây đó là việc redis sẽ thực thi các commands này lần lượt, tuần tự (1 command tại 1 thời điểm) từ đó giúp tránh được tình trạng dữ liệu không được cập nhật một cách chính xác do 2 commands cập nhật cùng 1 tài nguyên tại cùng 1 thời điểm (không toàn vẹn dữ liệu).
