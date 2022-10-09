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
