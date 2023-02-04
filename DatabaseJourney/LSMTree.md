# Bí mật phía sau NoSQL: LSMTree

Tham khảo từ [nguồn](https://www.youtube.com/watch?v=I6jB0nM9SKU)

## Với RDB truyền thống

Bản chất của RDB đó là việc sử dụng `B-Tree` cho việc lưu trữ dữ liệu. Các thao tác đọc với B-Tree diễn ra khá nhanh tuy nhiên thao tác ghi lại tốn nhiều chi phí và thời gian.

![Screen Shot 2023-02-04 at 18 07 24](https://user-images.githubusercontent.com/15076665/216759095-4f2cc3b4-fbb8-4d8b-b4d2-f03d675e0440.png)

## Với NoSQL

Chìa khoá ở đây chính là `memtable` - bản chất là `balanced tree` được lưu trong bộ nhớ

![Screen Shot 2023-02-04 at 18 10 06](https://user-images.githubusercontent.com/15076665/216759099-8a1eaeda-41eb-427c-9f77-b758cf712ffd.png)

Khi dữ liệu trong memtable đạt tới ngưỡng về kích cỡ, dữ liệu trong nó sẽ được ghi sang đĩa dưới dạng SSTable.

![Screen Shot 2023-02-04 at 18 13 36](https://user-images.githubusercontent.com/15076665/216759164-5d68413b-3936-4e68-874d-5aba53905470.png)

LSM-Tree:

![Screen Shot 2023-02-04 at 18 16 01](https://user-images.githubusercontent.com/15076665/216759266-c11852b7-50a9-4f46-808e-df6fd3b4bc30.png)
