# As const trong Typescript

Trong typescript việc khai báo như sau:

```TS
const UserType = {
  ADMIN: 'ADMIN',
  NORMAL: 'NORMAL',
} as const;
```

sẽ khiến cho các thuộc tính của const trở thành `readonly`. Nếu tiến hành gán giá trị mới cho thuộc tính của const thì compiler sẽ báo lỗi.

<img width="771" alt="Screen Shot 2022-08-11 at 22 54 38" src="https://user-images.githubusercontent.com/15076665/184150153-779c7872-f237-42a4-be52-8ec4bad7a3e0.png">

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
