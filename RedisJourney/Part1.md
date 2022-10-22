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

## Case study khi sử dụng redis để cache page với server side rendering

![File_000 (1)](https://user-images.githubusercontent.com/15076665/197318715-f351a6a9-a6f2-49c1-96bc-54b8e2b30bca.png)

## Database design methodology

### SQL DB design methodology

1. Đưa dữ liệu vào các bảng
2. Tìm cách query dữ liệu

### Redis design methodology

1. Tìm cách query dữ liệu
2. Cấu trúc dữ liệu để các câu queries có thể đưa ra response tối ưu và tốt nhất

#### Design considerations

- Loại dữ liệu sẽ lưu trữ ?
- Có cần cân nhắc về kích cỡ của dữ liệu ?
- Có cần thiết lập expire time cho dữ liệu ?
- Key naming policy là gì ?
- Business logic cần cân nhắc là gì ?

#### Key Naming

- Key phải là unique
- Những người đọc code phải hiểu được ý nghĩa của key
- Có thể sử dụng các generate function để tạo key nhằm tránh các lỗi typos
- Sử dụng `:` để ngăn cách các thành phần của key (VD: `user:45`, `items:19`, ...)
- Để search dễ dàng ta có thể sử dụng `#` trước unique ID (VD: `user#45`, `items#19`, `users:posts#100`) - đây là một tính năng khá mới của Redis

![File_000 (2)](https://user-images.githubusercontent.com/15076665/197325768-0d5a2ec8-a1c2-46e8-8923-ec9c5285499b.png)

## Hash in redis

### Lưu trữ

Hash trong redis sẽ có dạng như sau:

```json
{
  "name": "HEY",
  "created: 1919,
}
```

Tuy nhiên redis **KHÔNG CHO PHÉP** nested hash

```json
{
  "name": "HEY",
  "branches": {
    "main": "US",
    "sub-one": "JP",
  },
}
```

Và đồng thời redis cũng **KHÔNG CHO PHÉP** hash value là một array như dưới đây:

```json
{
  "name": "HEY",
  "branches": [ { "main": "US" }, { "sub-one": "JP" } ]
}
```

### Một vài commands cơ bản

`HSET`: set, tạo hash

```shell
hset company name "HEY" age 1915
```

`HGET`: get single value trong hash

```shell
hget company name
```

`HGETALL`: lấy toàn bộ key, value của hash

```shell
hgetall company
```

`HEXISTS`: kiểm tra key có tồn tại trong hash hay không:

- Nếu có trả vè `1`
- Nếu không trả về `0`

```shell
hexists company test
```

`DEL`: xoá hash

```shell
del company
```

`HDEL`: xoá một key-value pair duy nhất

```shell
hdel company name
```

`HINCRBY`: tăng giá trị value number trong hash (tạo mới key-value pair nếu chưa tồn tại)

```shell
hincrby company revenue 10
```

`HINCRBYFLOAT`: tăng giá trị theo số thực (real number)

```shell
hincrbyfloat company age 1.01
```

### Một vài chú ý liên quan đến hset và hgetall

![File_000 (3)](https://user-images.githubusercontent.com/15076665/197329587-580a3b3d-5388-49da-9131-54b5e421defd.png)

`hgetall` sẽ trả về một empty object `{}` thay vì `null` nếu ta truy vấn một key không tồn tại

### Khi nào nên và không nên dùng hash

#### Sử dụng hash khi

- Record có nhiều attributes
- Cần sort các records theo nhiều cách khác nhau
- Thường xuyên cần truy cập một single record

#### Không sử dụng hash khi

- Record sử dụng cho mục đích counting hoặc bắt buộc phải unique
- Record chỉ có 1 hoăc 2 attributes
- Cần tạo mối liên hệ giữa các records
- Time series data

## Một vài chú ý khi lưu trữ bằng hash

- Ta không cần thiết phải lưu ID trong hash nếu bản thân key tương ứng với toàn bộ hash đã mang tính duy nhất như trường hợp phía dưới đây

```json
{
  "user#1": {
    "id": 1,
    "name": "Haha"
  }
}
```

- Nếu ta truyền một `Date() object` vào Redis thì có thể redis sẽ lưu trữ date theo một format không như ý muốn

Tuy nhiên khi lấy dữ liệu ra ta lại muốn

- Có id trong dữ liệu trả về
- Một số dữ liệu thay vì là string sẽ là number
- Ngày tháng sẽ ở dạng `Date() object`

Để giải quyết những vấn đề trên ta sẽ sử dụng 2 methods là `serialize` và `deserialize`

![File_000 (4)](https://user-images.githubusercontent.com/15076665/197341082-73e8b394-7b05-4851-834f-9574c9f1942b.png)
