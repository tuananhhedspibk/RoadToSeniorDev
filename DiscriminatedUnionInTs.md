# Discriminated Union in TS

Ta xét ví dụ như sau:

```TS
interface Shape {
  kind: 'circle' || 'square';
  radius?: number;
  sideLength?: number;
}

function handleShape (shape: Shape) {
  // oops!
  if (shape.kind === "rect") {
    // ...
  }
}
/* * *
Hàm này sẽ gặp compile error do type của kind chỉ có thể là:
- "circle"
- "square"
* * */

function getArea (shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
// Object is possibly 'undefined' - error sẽ phát sinh khi compile
```

`strictNullChecks` sẽ đưa ra lỗi là vì `radius` và `sideLength` đều là `optional property`.

Ngay cả khi ta check kind của shape như dưới đây

```TS
function getArea(shape: Shape) {
  if (shape.kind === 'circle') {
    return Math.PI * shape.radius ** 2;
  }
}
// Object is possibly 'undefined' - error vẫn sẽ phát sinh khi compile
```

Nếu sử dụng `non-null asserttion` như sau:

```TS
function getArea(shape: Shape) {
  if (shape.kind === 'circle') {
    return Math.PI * shape.radius! ** 2;
  }
}
```

Vấn đề đã được giải quyết nhưng đây không phải là cách lí tưởng nhất vì vẫn có thể phát sinh lỗi khi code đi vào hoạt động do `optional properties` vẫn luôn được coi là "có" khi chúng ta đọc code.

Vấn đề cốt lõi ở đây đó là **Type checker không biết được liệu radius hoặc sideLength có tồn tại hay không khi dựa theo thuộc tính kind**. Do đó **chúng ta** cần nói với type checker rằng chúng ta **chắc chắn** `radius` hoặc `sideLength` luôn tồn tại.

```TS
interface Circle {
  kind: 'circle';
  radius: number;
}

interface Square {
  kind: 'square';
  sideLength: number;
}

type Shape = Circle | Square;
```

Ở đây ta chia `Shape definition` thành `Circle` và `Square` definition, với giá trị `kind` khác nhau đôi một và các thuộc tính `radius` và `sideLength` sẽ thành các `require properties`.

```TS
function getArea (shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
// Property 'radius' does not exist on type 'Shape'.
//   Property 'radius' does not exist on type 'Square'.
```

Lúc này `Shape` là union type và cũng có thể là `Square` mà `Square` thì không có `radius` nên lỗi sẽ phát sinh khi compile code.

Nếu ta check kind thì sao ?

```TS
function getArea (shape: Shape) {
  if (shape.kind === 'circle') {
    return Math.PI * shape.radius ** 2;
  }
}
```

Lỗi đã hết, nguyên nhân là do khi các types trong union type có chung thuộc tính literal type thì thuộc tính này sẽ được sử dụng để phân biệt các types bên trong union.

Thuộc tính chung này được gọi là `discriminant property`. Khi đó compiler sẽ thu hẹp phạm vi của type từ `Shape` xuống thành `Circle`.

Và đây được gọi là `Discriminated Union`.

Điều tương tự cũng xảy ra với cấu trúc `switch`, khi đó là không cần phải sử dụng `non-null assertion`

```TS
function getArea (shape: Shape) {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.sideLength ** 2;
  }
}
```
