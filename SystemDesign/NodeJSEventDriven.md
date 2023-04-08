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

Ở đây `WithLog` là một event emitter. Trong đó ta định nghĩa một instance function là `execute`, hàm này nhận đầu vào là một task function (dưới dạng tham số), nó được wrap bởi các log statements. Trước và sau khi thực hiện task function ta đều emit một event.

Tương ứng với các events đã emit như đã nói ở trên, ta cũng tiến hành đăng kí các listeners tương ứng với các events đó.

Kết quả đầu ra thu được sẽ như sau:

```txt
Before executing
About to execute
*** Executing task ***
Done with execute
After executing
```

Ta thấy rằng toàn bộ đầu ra ở trên đều được thực thi một các đồng bộ (synchronous). Đó là bởi vì tham số task function ở đây là synchronous function, nếu ta truyền vào đó một asynchronous function thì kết quả sẽ khác. Ta có thể mô phỏng bằng việc sử dụng hàm `setImmediate` như sau:

```JS
// ...
withLog.execute(() => {
  setImmediate(() => {
    console.log('*** Executing task ***');
  });
});
```

Kết quả đầu ra thu được sẽ như sau:

```txt
Before executing
About to execute
Done with execute
After executing
*** Executing task ***
```

Thứ tự này là sai, vậy ở đây ta cần phải emit event ngay sau khi một asynchronous function được thực thi xong, để làm được điều này ta sẽ tiến hành kết hợp giữa callback (hoặc promise) với event-based communication.

Một lợi ích của việc sử dụng event ở đây đó là với cùng một signal ta có thể "reaction" lại nhiều lần bằng việc đăng kí nhiều listener functions tương ứng với event đó. Trong khi đó với callback ta cần phải thêm nhiều logic vào một hàm callback duy nhất. Event cho phép ta có thể sử dụng nhiều external plugins để build các tính năng trên nền application core.

### Asynchronous Events

Ta cùng xét một ví dụ khác về asynchronous.

```JS
const fs = require('fs');
const EventEmitter = require('events');

class WithTime extends EventEmitter {
  execute(asyncFunc, ...args) {
    this.emit('begin');
    console.time('execute');
    asyncFunc(...args, (err, data) => {
      if (err) {
        return this.emit('error', err);
      }

      this.emit('data', data);
      console.timeEnd('execute');
      this.emit('end');
    });
  }
}

const withTime = new WithTime();

withTime.on('begin', () => console.log('About to execute'));
withTime.on('end', () => console.log('Done with execute'));

withTime.execute(fs.readFile, __filename);
```

Ở ví dụ trên, `WithTime` sẽ thực thi `asyncFunc` và đưa ra thời gian thực thi hàm đo với việc sử dụng `console.time` & `console.timeEnd`. Nó emit các events theo đúng thứ tự trước và sau khi thực thi.

Ở trên ta truyền vào hàm `fs.readFile`, đây là một hàm async, thay vì phải sử dụng callback để bắt được dữ liệu trả về của hàm này, ta có thể lắng nghe "sự kiện" data để có thể truy xuất được giá trị trả về của hàm.

Thực thi đoạn code trên, ta có kết quả đúng như mong đợi (theo đúng thứ tự)

```txt
About to execute
execute: 1.815ms
Done with execute
```

Nếu hàm async hỗ trợ Promise, ta có thể viết lại hàm như sau:

```JS
class WithTime extends EventEmitter {
  async execute(asyncFunc, ...args) {
    this.emit('begin');
    try {
      console.time('execute');
      const data = await asyncFunc(...args);
      this.emit('data', data);
      console.timeEnd('execute');
      this.emit('end');
    } catch (err) {
      this.emit('error', err);
    }
  }
}
```

### Events Arguments & Errors

Ở ví dụ phía trên, ta thấy có 2 events được emit đi kèm với các tham số khác.

Error event được emit với error object.

```JS
this.emit('error', err);
```

Data event được emit với data object.

```JS
this.emit('data', data);
```

Ta có thể sử dụng bao nhiêu tham số tuỳ thích ở phía sau `named event`, các tham số nay sẽ xuất hiện trong hàm listener mà chúng ta đã đăng kí tương ứng với `named event`.

```JS
withTime.on('data', (data) => {
  // do something with data
});
```

Ví dụ: với data event như ở trên, hàm listener sẽ nhận dữ liệu trả về của hàm `fs.readFile` như là một tham số đầu vào của nó.s

`Error event` là một trường hợp đặc biệt. Ở ví dụ về callback-based, nếu chúng ta không xử lí error event bằng một listener, node process sẽ tự động kết thúc (exit).

Để minh hoạ, ta sẽ tạo một lời gọi hàm thực thi khác với "argument tồi" như sau:

```JS
class WithTime extends EventEmitter {
  execute(asyncFunc, ...args) {
    console.time('execute');
    asyncFunc(...args, (err, data) => {
      if (err) {
        return this.emit('error', err); // Not Handled
      }

      console.timeEnd('execute');
    });
  }
}

const withTime = new WithTime();

withTime.execute(fs.readFile, ''); // BAD CALL
withTime.execute(fs.readFile, __filename);
```

Đoạn code trên sẽ đưa ra lỗi. Node process sẽ crash và tự động kết thúc

```txt
events.js:292
      throw er; // Unhandled 'error' event
      ^

Error: ENOENT: no such file or directory, open ''
```

Nếu chúng ta đăng kí một listener tương ứng với error event, thì kết quả của node process sẽ thay đổi. VD:

```JS
withTime.on('error', (err) => {
  // do something with err, for example log it somewhere
  console.log(err)
});
```

Nếu chúng ta làm như trên, thì node process không bị crash và tự động thoát, đồng thời các xử lí phía sau đó vẫn tiếp tục được thực thi.

```txt
[Error: ENOENT: no such file or directory, open ''] {
  errno: -2,
  code: 'ENOENT',
  syscall: 'open',
  path: ''
}
execute: 14.549ms
```

Một cách khác để xử lí các execeptions từ emitted errors đó chính là đăng kí một listener với global `uncaughtException` process event. Nhưng trên thực tế, catching error thông qua một global process là một ý tưởng tồi. Lời khuyên với `uncaughtException` đó là tránh dùng nó tối đa có thể, thế nhưng trong truòng hợp bất khả kháng (report về những gì đã xảy ra hoặc thực thi cleaup), chúng ta nên exit process bằng mọi cách.

```JS
process.on('uncaughtException', (err) => {
  // something went unhandled.
  // Do any cleanup and exit anyway!

  console.error(err); // don't do just that.

  // FORCE exit the process too.
  process.exit(1);
});
```

Hãy thử tưởng tượng, nhiều error events xảy ra cùng một lúc, điều này cũng có nghĩa rằng `uncaughtException` listener ở phía trên sẽ được chạy nhiều lần, dẫn đến vấn đề về cleaup.

`EventEmitter` module có đưa ra một method đó là method `once`, method này sẽ chỉ ra rằng, listener tương ứng với nó sẽ chỉ được chạy duy nhất một lần mà thôi. Đây là một cách sử dụng phổ biến với `uncaughtException` vì khi `uncaughtException` xảy ra lần đầu tiên ta sẽ tiến hành cleanup và thoát khỏi tiến trình luôn.

### Thứ tự của các listeners

Nếu chúng ta đăng kí nhiều listeners tương ứng với cùng một event, thì thứ tự thực hiện của các listeners sẽ giống với thứ tự được đăng kí của chúng.

```JS
withTime.on('data', (data) => {
  console.log(`Length: ${data.length}`);
});

withTime.on('data', (data) => {
  console.log(`Characters: ${data.toString().length}`);
});

withTime.execute(fs.readFile, __filename);
```

Kết quả thu được sẽ như sau:

```txt
Length: 1364
Characters: 1364
execute: 8.003ms
```

Đúng theo như thứ tự được đăng kí của các listeners ("Length" trước, "Characters" sau).

Thế nhưng nếu ta muốn, listener đăng kí sau được thực thi trước thì ta có thể sử dụng hàm `prependListener` như sau:

```JS
withTime.on('data', (data) => {
  console.log(`Length: ${data.length}`);
});

withTime.prependListener('data', (data) => {
  console.log(`Characters: ${data.toString().length}`);
});
```

Kết quả thu được sẽ như sau:

```txt
Characters: 1377
Length: 1377
execute: 8.985ms
```

"Characters" sẽ được thực thi trước "Lengths" do nó được đăng kí bằng `prependListener` function.

Cuối cùng để loại bỏ đi listener, ta có hàm `removeListener`.

Và đó là tất cả những gì tôi muốn trình bày với các bạn trong bài viết lần này, hẹn gặp lại ở các bài viết tiếp theo.
