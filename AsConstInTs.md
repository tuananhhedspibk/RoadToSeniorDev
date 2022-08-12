# as const trong Typescript

Trong typescript việc khai báo như sau:

```TS
const UserType = {
  ADMIN: 'ADMIN',
  NORMAL: 'NORMAL',
} as const;
```

sẽ khiến cho các thuộc tính của const trở thành `readonly`. Nếu tiến hành gán giá trị mới cho thuộc tính của const thì compiler sẽ báo lỗi.

<img width="771" alt="Screen Shot 2022-08-11 at 22 54 38" src="https://user-images.githubusercontent.com/15076665/184150153-779c7872-f237-42a4-be52-8ec4bad7a3e0.png">

## Khái niệm cơ bản về as const

```TS
const x = "hello" as const; // Type "hello"
```

`as const` còn được gọi là ** `constant assertion`. Khi sử dụng khai báo `as const` TS sẽ hiểu như sau:
- Sẽ không có sự mở rộng nào khác về kiểu (như đoạn code trên sẽ không có chuyện đi từ `"hello"` thành `string`)
- Object literal sẽ trở thành `readonly`

## Sự khác biệt giữa readonly và as const

```TS
type UserDetail = {
  name: string;
  age: number;
}

type User = {
  readonly id: number,
  readonly detail: UserDetail;
}

const user1: User = {
  id: 1,
  detail: {
    name: 'A',
    age: 10, 
  },
};

const user2 = {
  id: 1,
  detail: {
    name: 'A',
    age: 10, 
  },
} as const;
```

`readonly` chỉ có thể thiết lập cho thuộc tính mà thôi. Trong trường hợp thuộc tính cũng là 1 object thì các thuộc tính con của object vẫn có thể thay đổi được.

<img width="787" alt="Screen Shot 2022-08-11 at 23 03 46" src="https://user-images.githubusercontent.com/15076665/184151961-108c8d7f-12c8-4a32-b321-5a8dde39dc59.png">

<img width="627" alt="Screen Shot 2022-08-11 at 23 04 02" src="https://user-images.githubusercontent.com/15076665/184151968-cb1d2aee-5a57-4c88-a5f5-a0565df9d1a1.png">

<img width="745" alt="Screen Shot 2022-08-11 at 23 04 20" src="https://user-images.githubusercontent.com/15076665/184151970-accfac4a-4600-4308-b0f5-1c661679ec0b.png">

## So sánh giữa enum và as-const

1. Về mặt hiệu năng thì hoàn toàn như sau
2. Một vài sự khác biệt

```TS
enum Links = {
  Link1 = 'test',
  Link2 = 'test2',
};
```

Với định nghĩa như trên, ta sẽ thu được những thứ sau:

### Value với tên Links

Đây là object tồn tại ở run-time với thuộc tính là `Link1`, `Link2` với giá trị lần lượt là `test1`, `test2`

### Type với tên là Links

Type chỉ tồn tại ở compile-time. Ta có thể viết như sau:

```TS
interface Foo {
  link: Links
}
```

Đoạn code trên ý là: interface Foo phải có thuộc tính là `link` là `Links.Link1` hoặc `Links.link2`

### Namespace với tên là Links

Namespace này sẽ đem lại cho chúng ta 2 kiểu là `Links.Link1` và `Links.Link2`. Ví dụ ta có thể viết như dưới đây

```TS
interface Bar extends Foo {
  link: Links.Link1;
};
```

Với `as const` ta có thể mô tả lại những thứ trên như sau:

```TS
const Links = {
  Link1: 'test1',
  Link2: 'test2',
} as const;

type Links = (typeof Links)[keyof typeof Links];

namespace Links {
  export type Link1 = typeof Links.Link1;
  export type Link2 = typeof Links.Link2;
}
```

### Hơn thế nữa

```TS
const to: 'test1' = Links.Link1; // OK in anyway
const from: Links.Link1 = 'test1';
```

`enum` được coi là `nominal` subtype của `string` có nghĩa là:
- Từ `string` ta có thể ép xuống `Link.Link1`
- Từ `Link.Link1` không thể đẩy lên `string`  được

Vậy nếu ta muốn `Link.Link1` hoàn toàn `interchangable` với string literal thì `enum` sẽ **không phải** là một sự lựa chọn tốt ở đây.

Ngược lại nếu ta muốn cách duy nhất để có thể lấy được `test1` là phải thông qua `Link.Link1` thì nên sử dụng `enum`

> Tổng quan có nghĩa là nếu ta thực sự quan tâm đến string literal cụ thể thì không nên sử dụng enum còn nếu ta muốn sử dụng string literal như một gía trị "mờ mịt" thì nên sử dụng enum

```TS
enum Links {
  Link1 = '45454',
  Link2 = 'xxxyyyzzz',
}
```

Những chỗ sử dụng trực tiếp `test1` sẽ báo lỗi còn nếu sử dụng `Links.Link1` thì sẽ không báo lỗi
