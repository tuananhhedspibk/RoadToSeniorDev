# SQL for data science

## Data modeling

Là quá trình mô hình hoá thông tin thành các bảng có mối liên hệ với nhau, qua đó mô phỏng lại business model trong thực tếtế

## Relational vs Transactional Model

### Relational model

Cho phép query, insert, update, delete relational database một cách dễ dàng

### Transactional model

Operation DB

## Data model building block

Entiy, Attribute, Relationship

## Temporary table (bảng tạm)

Temporary table sẽ bị xoá đi khi session kết thúc. Nhanh hơn tạo real table.

Bảng tạm thường dùng cho các query phức tạp với subset hoặc join

```sql
CREATE TEMPORARY TABLE table_name AS (SELECT statement);
```

## Subqueries

Subqueries ở tầng sâu nhất sẽ được thực thi trước

### Best practice

Performance sẽ tồi khi số subqueries lồng nhau quá lớn. Subqueries select chỉ có thể retrieve được 1 column mà thôi.
