# Hiều về kiến trúc hướng sự kiện của Node.js

Bài viết được dịch từ [nguồn](https://medium.com/edge-coders/understanding-node-js-event-driven-architecture-223292fcbc2d)

Hầu hết các node objects như `HTTP request`, `HTTP response` hay `HTTP stream` - đều implement `EventEmitter` module nên chúng đều có thể:

- Emit event
- Lắng nghe event

Dạng thức đơn giản nhất của event-driven (hướng sự kiện) đó chính là `callback` trong các hàm có sẵn của NodeJS (VD: `fs.readFile`)

## Hãy gọi tôi khi bạn sẵn sàng, Node

Trước khi hỗ trợ promise và async/await thì cách mà Node xử lí các asynchronous events đó là **callback**.

Callback chỉ đơn thuần là một hàm được truyền vào một hàm khác như một tham số. Điều này trong JS là hoàn toàn có thể vì bản thân functions trong JS cũng được coi là các objects.

Một điều quan trọng đó là callback **KHÔNG PHẢI** là để chỉ asynchronous call trong code. Một hàm có thể gọi callback theo 2 cách:

- Đồng bộ (synchronous)
- Bất đồng bộ (Asynchronous)

Như ví dụ dưới đây, hàm `fileSize` nhận callback là tham số `cb`

```JS
function fileSize (fileName, cb) {
  if (typeof fileName !== 'string') {
    return cb(new TypeError('Argument should be string')); // Sync
  }

  fs.stat(fileName, (err, stats) => {
    if (err) { return cb(err); } // Async

    cb(null, stats.size); // Async
  });
}
```

Chú ý rằng, việc thiết kế sử dụng callback theo cả 2 cách *async* và *sync* là một **BAD PRACTICE**.

Ví dụ dưới đây, ta chỉ thuần tuý sử dụng callback theo cách *async* - *bất đồng bộ*

```JS
const readFileAsArray = function (file, cb) {
  fs.readFile(file, function (err, data) {
    if (err) {
      return cb(err);
    }

    const lines = data.toString().trim().split('\n');
    cb(null, lines);
  });
}
```

Hàm `readFileAsArray` sẽ có đầu vào là filepath, nó đọc nội dung file, cắt theo kí tự ngắt dòng và đưa ra một mảng dữ liệu, sau đó gọi đến hàm callback với mảng dữ liệu đó.

Nếu ta có một task đó là đến số lượng các số lẻ có trong file với nội dung như dưới

```txt
10
11
12
13
14
15
```

thì ta có thể sử dụng `readFileAsArray` để đơn giản hoá code như sau:

```JS
readFileAsArray('./numbers.txt', (err, lines) => {
  if (err) throw err;

  const numbers = lines.map(Number);
  const oddNumbers = numbers.filter(n => n%2 === 1);
  console.log('Odd numbers count: ', oddNumbers.length);
});
```

Ở đây node callback style được sử dụng một cách đúng đắn. Hàm callback sẽ nhận tham số thứ nhất là một object `err` (nullable), chúng ta sẽ truyền callback như tham số cuối cùng của host function.

Hãy nhớ:

> Truyền callback như một tham số cuối của host function, callback phải luôn luôn nhận tham số đầu tiên là một object lỗi

## Javascript hiện đại thay thế callbacks

JS hiện đại có thêm promise như một công cụ thay thế cho callback
