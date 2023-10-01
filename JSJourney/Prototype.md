# Ghi chép nhanh về Prototype trong JS

## Những khái niệm cơ bản

Là một khái niệm core về OOP của JS, nó chính là khung xương cho khái niệm kế thừa trong OOP.

Trong JS mọi thứ đều được coi là Object, prototype là một thuộc tính có sẵn của mọi object và bản thân prototype cũng là một object.

Nó giống như meta-data của object đó.

Xét đoạn code sau:

```js
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

/*
 * Khi Person được tạo, prototype của nó chỉ chứa một thuộc tính duy nhất là constructor
 * thuộc tính này sẽ trỏ ngược lại vào function Person
 * Khi object được tạo ra bằng cách new Person("firstName", "lastName")
 * thì nó sẽ kế thừa mọi thuộc tính của Person.prototype
 */
```

Ta cũng có thể thêm một hàm con khác vào Person trên như sau:

```js
Person.prototype.showFullName = function () {
  console.log(this.firstName + " " + this.lastName);
};
```

Các object trong JS cũng được xây dựng dựa theo nguyên lí tương tự như vậy, các object được tạo ra bằng `new Object()` hoặc `{}` sẽ có prototype là `Object.prototype`

## Prototype chain

Là một tính năng khá hay khác của prototype. Lấy ví dụ như sau

```js
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

const person1 = new Person("firstName");

// Khi ta truy cập vào thuộc tính lastName: person1.lastName
// ta thấy rằng, person1 không có thuộc tính này
// Do khi được tạo ra, bản thân person1 cũng có một thuộc tính prototype "giả" là __proto__
// nên khi truy cập vào lastName, quá trình JS Engine tìm kiếm sẽ như sau
// person1.__proto__  → Person.prototype
// Tức là nếu không thấy trong person1.__proto__ thì sẽ tìm lên Person.prototype
```

```js
var obj1 = {a: 1};
var obj2 = Object.create(obj1);

obj2.__proto__; // → { a: 1 }
```

Ở đoạn code trên, ta thấy rằng `obj2` được tạo ra dựa trên prototype của `obj1` nên theo như cơ chế protoype chain, `obj2` sẽ chứa thuộc tính `a` được kế thừa từ `obj1`

## Tài liệu tham khảo

<https://viblo.asia/p/prototype-trong-javascript-RQqKLzOrl7z>
