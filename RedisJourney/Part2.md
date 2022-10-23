# Redis Basic - Part 2

## Powerful Design Pattern

### Datetime format

Với các format như:

- `1994-11-05T08:15:30-05:00`
- `Thu Mar 17 1994, 10:56:16 GMT-0500`

thì mặc định redis sẽ không thể search hoặc sort chúng được

Với các format như:

- 10123139123213: `Unix time` giây - tính từ `1970/01/01`
- 10123139123213000: `Unix time` mili giây

thì redis có thể search và sort được

Do đó hãy chuyển các `Date` hoặc `DateTime` object thành `unix time second` hoặc `unix time milisecond`.

> Redis không tự động generate ID

### Batching commands with pipeline

Khi tiến hành thực thi nhiều commands cùng một lúc, ta có thể sử dụng pipeline để tạo thành "một command khổng lồ" từ các commands thành phần

## Basic of set

### sadd & smembers

`SADD`: thêm giá trị vào set

```shell
sadd colors red
```

`SMEMBERS`: trả về toàn bộ phần tử bên trong set

```shell
smembers colors
```

### Union of set

`Union`: trả về toàn bộ các elements khác nhau lấy từ tất cả các sets vào một set duy nhất

```shell
sunion set1 set2 ... setn
```

### Intersect of set

Trả về các phần tử tồn tại ở mọi set

```shell
sinter set1 set2 ... setn
```

### Diff of set

Trả về các phần tử chỉ tồn tại ở set đầu tiên và không tồn tại ở các sets còn lại

```shell
sdiff set1 set2 ... setn
```

### Store with variation

```shell
sinterstore result set1 set2 ... setn
```

Lưu kết quả của `sinter` vào result

### is member

Trả về 0, 1 nếu phần tử "không thuộc" hoặc "thuộc" set

```shell
sismember set ele
```

`smismember` sẽ chạy `sismember` nhiều lần cùng một lúc (kiểm tra xem các phần tử có thuộc set hay không)

```shell
smismember set ele1 ele2 ... elen
```

### Scan set

`SCARD`: trả về số lượng member của set

```shell
scard colors
```

### Remove element from set

`SREM`: loại bỏ element trong set

```shell
sremove colors blue
```

### Scan all element in set

```shell
sscan colors 0 COUNT 2
```

0: cursor ID (tương tự như khái niệm cursor trong khi đọc file)
COUNT 2: số lượng phần tử sẽ trả về

`sscan` cho phép chúng ta có thể quét qua các phần tử trong set theo từng phần (piece by piece). Do set không có tính thứ tự tuần tự như mảng thông thường nên thứ tự trả về của `sscan` sẽ khác với `smembers`

Hơn nữa giá trị trả về của `sscan` sẽ có format như sau:

```json
{
  cursorId,
  [
    ele1,
    ele2,
    ...,
    elen
  ]
}
```

Trong đó cursorId sẽ là vị trí tiếp theo của cursor dùng để duyệt set. Nói một cách đơn giản `sscan` tron Redis giống như cơ chế phân trang trong web vậy.

`sscan` sẽ sử dụng với các set lớn (khi việc duyệt toàn bộ set là vô cùng khó khăn)

### Set usecase

- Khi có yêu cầu unique với các values
- Tạo relationship giữa các records
- Tìm điểm chung về attrs giữa các records khác nhau
- Lists các elements mà không cần phải quan tâm đến thứ tự
