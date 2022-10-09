# Type vs Interface trong Typescript

Dịch tóm tắt từ [nguồn](https://zenn.dev/luvmini511/articles/6c6f69481c2d17)

```TS
let value: number = 10;

value = "10"; // TS sẽ báo lỗi type error
```

Cách định nghĩa trên được gọi là `type annotation`, tuy nhiên với trường hợp của object thì sao

```TS
const apple: { nickName: string; isHuman: boolean; level: number } = {
  nickName: 'ABC',
  isHuman: true,
  level: 0,
};
```

Nhìn qua thôi cũng thấy khá là cực.

Có thể dùng interface để xử lí case trên. Sử dụng interface giống như việc **đặt tên cho một object type**

```TS
interface Member {
  nickName: string;
  isHuman: boolean;
  level: number;
}
```

Ngoài ra cũng có thể dùng `type`

```TS
type Member = {
  nickName: string;
  isHuman: boolean;
  level: number;
}
```

Sự khác biệt nằm ở chỗ

> interface sẽ định nghĩa một kiểu mới còn type sẽ gán tên cho một kiểu đã có trước đó và không có tên nhằm mục đích tham chiếu đến kiểu đó

```TS
interface Member {
  nickName: string;
  isHuman: boolean;
  level: number;
} // Định nghĩa một kiểu mới


type Member = {
  nickName: string;
  isHuman: boolean;
  level: number;
} // gán tên Member cho kiểu { nickName: string; isHuman: boolean; level: number; } - lúc ban đầu chưa có tên
```

## Các loại type có thể định nghĩa

`interface` chỉ có thể định nghĩa kiểu cho:

- Class
- Object

`type` có thể định nghĩa kiểu theo cho nhiều đối tượng

```TS
type Color = 'Red' | 'White' | 'Black';
let color: Color = 'White';

color = 'Green'; // Type 'Green' is not assignable to type 'Color'.
```

## Khả năng mở rộng

```TS
interface User {
  name: string;
}

interface User {
  level: number;
}

const user: User = {
  name: 'A',
  level: 1,
};
```

`Interface` có thể mở rộng được, nhìn qua đoạn code phía trên ta sẽ nghĩ rằng mình đang định nghĩa lại interface User nhưng không ta đã thêm vào định nghĩa của nó thuộc tính `level`

`Type` thì không có khả năng mở rộng

```TS
type User = {
  name: string;
}; // Duplicate identifier 'User'

type User = {
  level: number;
}; // Duplicate identifier 'User'
```

## Vậy nên theo trường phái nào

### Với interface

Ta có thể thấy đó là khả năng mở rộng, tuy nhiên nếu trong một file code có cả nghìn dòng thì việc mở rộng đôi khi lại đem đến điều bất lợi

```TS
interface Shoes {
  size: number;
}

const shoes: Shoes = { size: 10 }; // Property 'isSecondhand' is missing in type '{ size: number; }' but required in type 'Shoes'
```

Nguyên nhân là bởi ở một chỗ nào đó trong code ta đã tiến hành định nghĩa như sau:

```TS
interface Shoes {
  isSecondHand: boolean;
}
```

Do đó thuộc tính `isSecondHand` cần phải được set giá trị

### Với type

Đúng là nó không có khả năng mở rộng nhưng cách viết dưới đây lại hoàn toàn ngược lại

```TS
type ErrorHandling = {
  success: boolean;
  error?: { message: string };
};

type ArtistsData = {
  artists: { name: string }[];
};

type ArtistsResponse = ArtistsData & ErrorHandling;

const dummyData: ArtistsResponse = {
  success: true,
  artists: [{ name: "apple" }, { name: "banana" }],
};
```

Với interface ta sẽ có cách viết như sau

```TS
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
};

interface ArtistsData {
  artists: { name: string }[];
};

interface ArtistsResponse extends ArtistsData, ErrorHandling {}

const dummyData: ArtistsResponse = {
  artists: [{ name: "apple" }, { name: "banana" }],
  success: true
};
```
