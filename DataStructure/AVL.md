# AVL Tree

Bài viết được dịch từ [nguồn](https://www.growingwiththeweb.com/data-structures/avl-tree/overview/)

AVL Tree (Cây AVL) là cấu trúc dữ liệu tương tự như cây đỏ đen tự cân bằng - [self-balancing red-black tree](https://www.growingwiththeweb.com/data-structures/red-black-tree/overview/) nhưng thay vì sử dụng "màu" cho BST để có thể đảm bảo sự cân bằng của BST thì AVL sẽ theo dõi độ cao của cây để đảm bảo tính cân bằng cho cây

## Độ phức tạp:

**Delete**: xoá một node với key cho trước, độ phức tạp là `O(logn)`.
**Insert**: thêm một node với key, độ phức tạp là `O(logn)`.
**Search**: tìm kiếm một node với key cho trước, độ phức tạp là `O(logn)`.

## Đặc điểm

Cây AVL là BST với đặc điểm sau:

- Mỗi một node con sẽ có chênh lệch độ cao giữa cây con trái và cây con phải không quá 1

```matlab
|hl - hr| <= 1
```

## Biểu diễn

Có nhiều cách để biểu diễn BST. Trong đó cách thứ nhất đó là biểu diễn dưới dạng node, tham chiếu tới 2 nodes con như hình minh hoạ bên dưới:

![Screen Shot 2023-02-18 at 18 57 59](https://user-images.githubusercontent.com/15076665/219854181-fd97aab3-cc8e-48d6-95de-8de45c5345f4.png)

Cách thứ 2 là dùng mảng với `2i + 1` là index của node con trái, `2i + 2` là index của node con phải, trong đó `i` là index của node cha. Xem hình minh hoạ bên dưới:

![Screen Shot 2023-02-18 at 19 00 24](https://user-images.githubusercontent.com/15076665/219854276-8f16aae1-5a85-4090-ab2f-25280e96f93e.png)

Ngoài ra index của node cha sẽ được tính theo công thức `[(i - 1) / 2]` - với `[]` ở đây là phép làm tròn xuống.

## Độ cao của node

Độ cao của một node chính là số bước để đi từ node đó xuống node lá tận cùng của các cây con gắn với node đó. Ta có công thức đệ quy để tính độ cao của node như sau:

```matlab
h = max(hl, hr) + 1
```

Chú ý rằng: độ cao của node lá là: 0

> Khái niệm về "độ cao" của node ngược lại với "độ sâu" của node, "độ sâu" của một node chính là số bước tính từ node gốc đến node đó

Hình minh hoạ cho khái niệm "độ sâu" của node:

![Screen Shot 2023-02-18 at 19 07 55](https://user-images.githubusercontent.com/15076665/219855015-7c2151b4-fd33-4cf2-9f25-3e368dd05299.png)

## Thao tác cân bằng cây

Khi insert hoặc delete node, ta có thể sẽ phải "cân bằng" lại cây để đảm bảo độ cao cây con trái và phải của mọi nodes không lệch nhau quá 1. Ta có 4 trườngh hợp cả thảy cần phải cân bằng như dưới đây.

### Left Left

Nghĩa là trường hợp độ cao cây con bên trái lớn hơn cây con bên phải 2 đơn vị. Bản thân cây con trái cũng có thể là "cân bằng" hoặc "lệch trái". Có thể cân bằng bằng cách "xoay phải" tại node mất cân bằng.

![Screen Shot 2023-02-18 at 19 22 04](https://user-images.githubusercontent.com/15076665/219855231-9be370b9-98bb-4655-86e1-cb473b2f93fb.png)

### Left right

Đây là trường hợp cây con trái cao hơn cây con phải 2 đơn vị và cây con trái bị lệch phải. Ta giải quyết bằng cách "xoay trái" cây con trái, lúc này bài toán lại quay về trường hợp `left left` như trên, nên sau đó ta sẽ tiến hành "xoay phải" tại node mất cân bằng.

![Screen Shot 2023-02-18 at 19 24 59](https://user-images.githubusercontent.com/15076665/219855381-e730fb94-dd01-4a2f-8847-3ee2c67be214.png)

### Right right

Đây là trường hợp mà cây con phải cao hơn 2 đơn vị so với cây con trái và cây con phải bị "lệch phải". Ta giải quyết bằng cách "xoay trái" ở node mất cân bằng.

![Screen Shot 2023-02-18 at 19 26 57](https://user-images.githubusercontent.com/15076665/219855438-ca8e2f3f-0c06-45d2-af07-763c69684596.png)
