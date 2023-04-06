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

JS hiện đại có thêm promise như một công cụ thay thế cho callback để chạy gọi các async APIs. Thay vì truyền callback như một tham số s, tiến hành xử lí lỗi và xử lí kết quả thành công ở cùng một chỗ thì với promise ta có thể tách hai xử lí lỗi và xử lí thành công thành 2 blocks tách biệt nhau. Không những thế promise còn cho phép chúng ta có thể gọi một chuỗi các lời gọi async thay vì lồng chúng lại với nhau.

Nếu hàm `readFileAsArray` hỗ trợ promise, chúng ta có thể sử dụng nó như sau:

```JS
readFileAsArray('./numbers.txt')
  .then(lines => {
    const numbers = lines.map(Number);
    const oddNumbers = numbers.filter(n => n % 2 === 1);
    console.log('Odd numbers count: ', oddNumbers.length);
  })
  .catch(console.error);
```

Ta thấy ở đây hàm `.then` sẽ là nơi ta tiến hành xử lí mảng lines tương tự như những gì đã làm với callback version, và chúng ta sẽ xử lí lỗi trong lời gọi hàm `.catch`, hàm này sẽ giúp chúng ta có thể truy cập vào error object khi lỗi xảy ra.

Việc làm cho host function hỗ trợ promise interface trở nên dễ dàng hơn trong Javascript hiện đại với **Promise object**. Ở đây `readFileAsArray` function sẽ được thay đổi để hỗ trợ promise interface.

```JS
const readFileAsArray = function(file, cb = () => {}) {
  return new Promise((resolve, reject) => {
    fs.readFile(file, function (err, data) {
      if (err) {
        reject(err);
        return cb(err);
      }

      const lines = data.toString().trim().split('\n');
      resolve(lines);
      cb(null, lines);
    });
  });
}
```

Ta sẽ wrap `fs.readFile` async call bằng một Promise object, object này nhận 2 tham số là `resolve` và `reject`. `reject` function sẽ dùng để gọi khi lỗi xảy ra, trong khi đó ta sẽ gọi callback với dữ liệu bằng `resolve` function.

### Xử lí promises với async/await

Với callback ta cần lồng nhiều lớp hàm khi cần xử lí nhiều async functions. Còn với promise thì việc này lại không quá khó khăn.

Ta có thể xử lí hàm `readFileAsArray` với async/await như sau:

```JS
async function countOdd () {
  try {
    const lines = await readFileAsArray('./numbers');
    const numbers = lines.map(Number);
    const oddCount = numbers.filter(n => n%2 === 1).length;
    console.log('Odd numbers count:', oddCount);
  } catch(err) {
    console.error(err);
  }
}
countOdd();
```

Chúng ta sẽ tạo một `async function` với từ khoá `async` ở phía trước, chúng ta gọi `readFileAsArray` nhưng với từ khoá `await` ở trước, các xử lí tiếp theo sẽ được viết tương tự như synchronous function.

## EventEmitter Module

Đây là module core của kiến trúc hướng sự kiện (event-driven architecture) bên trong Node. Nó cho phép các objects có thể tương tác qua lại với nhau. Rất nhiều module có sẵn của node kế thừa từ EventEmitter.

Concept ở đây rất đơn giản: emitter objects sẽ đẩy đi (emit) các named events, các events này sẽ làm cho các listeners đã đăng kí trước đó được gọi. Do đó một emitter object về cơ bản có 2 tính năng chính:

- Emit named events.
- Registering & unregistering listener functions.

Để làm việc với EventEmitter ta chỉ cần tạo một class kế thừa `EventEmitter` class có sẵn của Node.

```JS
class MyEmitter extends EventEmitter {

}

// emitter object chính là thể hiện được tạo ra từ EventEmitter-based classes:

const myEmitter = new MyEmitter();

// Đẩy đi (emit) sự kiện như sau:

myEmitter.emit('something-happended');
```

Emit một event có thể là tín hiệu cho thấy có một điều gì đó hoặc một sự kiện nào đó vừa mới xảy ra. Sự kiện này thường là state change event trong emitting object.

Ta có thể sử dụng hàm `on()` để thêm listener function cho emitting object. Listener functions này sẽ được gọi khi emitting object emit event tương ứng.

### Event !== Asynchrony

```JS
const EventEmitter = require('events');

class WithLog extends EventEmitter {
  execute(taskFunc) {
    console.log('Before executing');
    this.emit('begin');
    taskFunc();
    this.emit('end');
    console.log('After executing');
  }
}

const withLog = new WithLog();

withLog.on('begin', () => console.log('About to execute'));
withLog.on('end', () => console.log('Done with execute'));

withLog.execute() => console.log('*** Executing task ***'));
```
