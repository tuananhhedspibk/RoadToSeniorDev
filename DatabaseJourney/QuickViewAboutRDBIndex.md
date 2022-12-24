# Quick view about RDB index

## Index là gì, có tác dụng gì

Index trong RDS table là một `schema object`. Nó đóng vai trò tăng tốc các câu `SELECT`, truy xuất dữ liệu và giảm `I/O` input/output.

Tuy nhiên nó có một nhược điểm đó là làm chậm đi quá trình `INSERT`, `UPDATE` dữ liệu. Bản thân việc thêm, xoá index **KHÔNG HỀ LÀM ẢNH HƯỞNG ĐẾN DỮ LIỆU** của table.

## Khi nào nên và không nên dùng index

### Nên dùng index khi

- Bảng có kích cỡ lớn, column có khoảng dữ liệu rộng
- Column không bao gồm một số lượng lớn các `null` values
- Một hoặc một vài columns thường được sử dụng trong các mệnh đề `WHERE` hoặc `JOIN`

### Không nên dùng index khi

- Bảng nhỏ
- Column không thường xuyên được sử dụng trong các mệnh đề `WHERE` hoặc `JOIN`
- Column được update thường xuyên

## Đào sâu về index trong DB

### Khái niệm cơ bản về index

Index trong DB có thể hiểu là một cấu trúc dữ liệu giúp map các search keys với dữ liệu tương ứng trong ổ đĩa.

Thông thường, index sẽ gắn với một hoặc nhiều cột hay xuất hiện trong các mệnh đề `WHERE` truy vấn dữ liệu.

Nếu không có index, câu truy vấn sẽ quét qua toàn bộ dữ liệu, filter và trả về kết quả, từ đó dẫn đến tình trạng hiệu năng câu truy vấn rất thấp.

Thử tạo một bảng với cấu trúc như sau:

```sql
CREATE TABLE index_demo ( 
  name VARCHAR(20) NOT NULL, 
  age INT, 
  pan_no VARCHAR(20), 
  phone_no VARCHAR(20) 
);
```

Chạy lệnh sau để biết thông tin về index của bảng:

```sql
SHOW INDEX FROM index_demo;
```

Chạy câu lệnh dưới để biết rằng khi không có index thì `SELECT` query sẽ chạy như thế nào

```sql
EXPLAIN SELECT * FROM index_demo WHERE name = 'alex';
```

Kết quả:

- `possible_keys`: chỉ ra những indices nào sẽ được sử dụng câu query.
- `key`: chỉ ra index nào sẽ thực sự được sử dụng.

### Những điều cần cân nhắc khi tạo primary key

- Primary key nên thường xuyên được sử dụng trong các câu query.
- Primary key dùng để phân biệt các dòng trong bảng, nếu primary key dùng cho nhiều cột thì "tổ hợp" các cột phải có khả năng phân biệt các dòng trong bảng.
- Cột dùng làm primary key `KHÔNG ĐƯỢC PHÉP NULL` vì giá trị của cột này phải ở dạng so sánh được. Trong SQL thì `NULL` được hiểu là `undefined`.
- Primary key lí tưởng nhất nên là `INT` hoặc `BIGINT` vì bản thân các giá trị kiểu này sẽ dễ so sánh hơn và câu query cũng sẽ duyệt qua các dòng nhanh hơn.

> Trong hầu hết các DB thì khi ta đặt một cột là primary key thì mặc định primary index sẽ được đánh trên cột đó.

### Khác biệt giữa key và index

Khái niệm key thường chỉ nằm ở mức "cột" (dùng để phân biệt các dòng với nhau) trong khi index là một cấu trúc dữ liệu có tầm ảnh hưởng đến việc truy vấn dữ liệu trên toàn bộ bảng.

### Điều gì sẽ xảy ra nếu không tạo primary key ?

Việc tạo primary key là không bắt buộc khi tạo bảng, tuy nhiên InnoDB bắt buộc phải có primary key cho từng bảng nên do đó, trong trường hợp ta không tự tạo primary key cho bảng thì InnoDB sẽ tự động tạo primary key cho bảng đó.

Nếu ta thêm primary key cho bảng thì InnoDB sẽ tự động xoá key mà nó tự tạo cho bảng trước đó

```sql
SHOW EXTENDED INDEX FROM index_demo;
```

`EXTENDED` sẽ đưa ra các indices không thể sử dụng nhưng lại được quản lí bởi MySQL.

```sql
SHOW INDEXES FROM index_demo;
```

- **Collation**: Thứ tự sắp xếp, `A`: ascending, `D`: descending, `NULL`: không được sắp xếp.
- **Cardinality**: số lượng unique values trong index

```sql
EXPLAIN SELECT * FROM index_demo WHERE phone_no = '90123123213123';
```

Chạy câu query trên ta thu được kết quả như sau:

![Screen Shot 2022-12-24 at 19 35 01](https://user-images.githubusercontent.com/15076665/209432180-3f9b3f2c-e28d-42a4-8b5e-f7d1be7846c3.png)

Số lượng row tác động ở đây chỉ là `1` nên do đó ta thấy index có tác dụng thu hẹp phạm vi mà câu query chạy trên đó.

### Clustered Index

## Tài liệu tham khảo

- https://medium.com/free-code-camp/database-indexing-at-a-glance-bb50809d48bd
