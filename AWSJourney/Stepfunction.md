# Sử dụng AWS CDK để tạo Stepfunction, StateMachine

<https://dev.classmethod.jp/articles/aws-step-functions-for-beginner/>

## Khái niệm về Stepfunction

Là một serverless service cho phép build các workflows, các workflows này được gọi là **StateMachine**.

Bản chất là tập hợp của các states hoặc cũng có thể coi là StateMachine là một chuỗi các **event-driven steps**

Hiểu một cách đơn giản thì StateMachine "kiểu kiểu" như thế này

![State Machine, Step function Note](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/1e265b53-751c-4add-bbd2-40b90616cd7a)

## ASL (Amazon States Language)

ASL là một ngôn ngữ (gần giống JSON) để định nghĩa ra StateMachine.

Ta lấy ví dụ:

```JSON
{
  "Comment": "Example",
  "StartAt": "OpenState",
  "States": {
    "OpenState": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-northeast-1:11111111:function:OPEN_FUNCTION",
      "Next": "ChoiceState"
    },
    "ChoiceState": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.foo",
          "NumbericEquals": 10,
          "Next": "FirstMatchChoice"
        },
        {
          "Variable": "$.foo",
          "NumericalEquals": 20,
          "Next": "SecondMatchChoice"
        }
      ],
      "Default": "DefaultChoice"
    },
    "FirstMatchChoice": {
      "Type" : "Task",
      "Resource": "arn:aws:lambda:ap-northeast-1:11111111:function:FIRST_MATCH_CHOICE",
      "Next": "NextState"
    },
    "SecondMatchChoice": {
      "Type" : "Task",
      "Resource": "arn:aws:lambda:ap-northeast-1:11111111:function:SECOND_MATCH_CHOICE",
      "Next": "NextState"
    },
    "DefaultState": {
      "Type": "Fail",
      "Error": "DefaultStateError",
      "Cause": "No Matches!"
    },
    "NextState": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-northeast-1:11111111:function:FUNCTION_NAME",
      "End": true
    }
  }
}
```

Giải thích sơ qua về một vài từ khoá:

- "StartAt": State đầu tiên sẽ được thực hiện
- "States": Danh sách các states có trong StateMachine

Các loại State:

- "Task": một xử lí đơn thuần
- "Pass": đơn thuần truyền dữ liệu
- "Choice": điều kiện rẽ nhánh
- ...

![State Machine Step function Note (1)](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/ab61993d-c301-492b-a337-dcdd35df1f41)

## Sử dụng AWS CDK để tạo một stepfunction đơn giản

### AWS CDK là gì? Có khác gì so với AWS SDK

CDK là công cụ được cung cấp bởi AWS với mục đích tạo các AWS resource thông qua code (Infrastructure As Code - IAC)

SDK (Software Development Kit) là thư viện dùng để tương tác với AWS resource.

### Stepfunction với CDK

Bạn đọc có thể tham khảo toàn bộ source code của tôi tại [đây](https://github.com/tuananhhedspibk/StepFunctionPractice)

Để tiến hành deploy lên AWS bạn đọc cần chạy hai câu lệnh

```shell
npm run cdk bootstrap
npm run cdk deploy
```

Hình dưới đây sẽ mô tả sơ qua về cấu trúc của stepfunction:

![Screen Shot 2023-10-25 at 22 38 31](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/ed659034-245d-4833-b4f8-d238cc667fad)
_Hình-1_

State đầu tiên đó là `Submit Job`, kế tiếp sẽ là `Wait X Seconds`, sau đó là `Get Job Status` và `Job Complete`.

`Job Complete` sẽ đóng vai trò như một điều kiện rẽ nhánh:

- Hoặc là quay về `Wait X Seconds`.
- Hoặc là đi đến `Job Failed`
- Hoặc là đi đến `Get Final Job Status`

Hai states `Job Failed` & `Get Final Job Status` sẽ có điểm kết thúc chính là kết thúc của stepfunction `end`.

#### Phân tích từ code

Đầu tiên ta định nghĩa hai hàm lambda (CheckLambda & SubmitLambda)

```ts
const checkLambda = new lambda.Function(this, "CheckLambda", {
  code: new lambda.InlineCode(
    fs.readFileSync("lib/lambdas/check_status.py", {encoding: "utf-8"})
  ),
  handler: "index.main",
  timeout: cdk.Duration.seconds(30),
  runtime: lambda.Runtime.PYTHON_3_9,
});

const submitLambda = new lambda.Function(this, "SubmitLambda", {
  code: new lambda.InlineCode(
    fs.readFileSync("lib/lambdas/submit.py", {encoding: "utf-8"})
  ),
  handler: "index.main",
  timeout: cdk.Duration.seconds(30),
  runtime: lambda.Runtime.PYTHON_3_9,
});
```

Sau khi deploy xong ta sẽ thu được kết quả là hai hàm lambda tương ứng được tạo ra trên AWS console

![Screen Shot 2023-10-25 at 22 11 36](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/301da7f3-d97e-46b5-9798-bb925d46ccc1)
_Hình-2_

![Screen Shot 2023-10-25 at 22 11 54](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/41833ccd-b827-4e7a-91bf-58956c686162)
_Hình-3_

Kế đến ta bắt đầu định nghĩa từng state cho stepfunction.

Đầu tiên là **Submit Job**

```ts
const submitJob = new tasks.LambdaInvoke(this, "Submit Job", {
  lambdaFunction: submitLambda,
  outputPath: "$.Payload",
});
```

State này sẽ có nhiệm vụ gọi đến hàm `SubmitLambda`.

ASL thu được sẽ như sau:

```JSON
{
  "Submit Job": {
    "Type": "Task",
      "OutputPath": "$.Payload",
    "Resource": "arn:aws:states:::lambda:invoke",
    "Parameters": {
      "FunctionName": "arn:aws:lambda:ap-northeast-1:265593583957:function:JobPollerStack-SubmitLambda8054545E-frfXqniU6RTT",
      "Payload.$": "$"
    }
  },
}
```

ta có thể thấy rõ được việc State **SubmitJob** có quyền **lambda:invoke - gọi đến hàm lambda** và hàm lambda được gọi sẽ có ARN là **arn:aws:lambda:ap-northeast-1:265593583957:function:JobPollerStack-SubmitLambda8054545E-frfXqniU6RTT**

Tiếp đến sẽ là **Wait X Seconds**

```ts
const waitX = new sfn.Wait(this, "Wait X Seconds", {
  time: sfn.WaitTime.duration(cdk.Duration.seconds(30)),
});
```

Ở đây state thu được chỉ thuần tuý là một "Wait State" với chức năng "chờ" trong 30s với ASL tương ứng như sau:

```JSON
{
  "Wait X Seconds": {
    "Type": "Wait",
    "Seconds": 30,
    "Next": "Get Job Status"
  },
}
```

Kế đến sẽ là **Get Job Status** state

```ts
const getStatus = new tasks.LambdaInvoke(this, "Get Job Status", {
  lambdaFunction: checkLambda,
  outputPath: "$.Payload",
});
```

State này sẽ gọi đến hàm **CheckLambda** đã được định nghĩa trước đó.

ASL tương ứng như sau:

```JSON
{
  "Get Job Status": {
    "Next": "Job Complete ?",
    "Type": "Task",
    "OutputPath": "$.Payload",
    "Resource": "arn:aws:states:::lambda:invoke",
    "Parameters": {
      "FunctionName": "arn:aws:lambda:ap-northeast-1:265593583957:function:JobPollerStack-CheckLambda9CBBF9BA-Hys585A5SJk7",
      "Payload.$": "$"
    }
  }
}
```

Sau đó sẽ là **Job Failed** state

```ts
const jobFailed = new sfn.Fail(this, "Job Failed", {
  cause: "AWS Batch Job Failed",
  error: "DescribeJob returned FAILED",
});
```

Đây chỉ thuần tuý là một state failed (biểu thị cho một kết quả "thất bại") mà thôi.

ASL thu được

```JSON
{
  "Job Failed": {
    "Type": "Fail",
    "Error": "DescribeJob returned FAILED",
    "Cause": "AWS Batch Job Failed"
  },
}
```

Cuối cùng sẽ là **Get Final Job Status** state.

```ts
const finalStatus = new tasks.LambdaInvoke(this, "Get Final Job Status", {
  lambdaFunction: checkLambda,
  outputPath: "$.Payload",
});
```

Nó sẽ gọi đến **CheckLambda** function với ASL như sau:

```JSON
{
  "Get Final Job Status": {
    "End": true,
    "Type": "Task",
    "OutputPath": "$.Payload",
    "Resource": "arn:aws:states:::lambda:invoke",
    "Parameters": {
      "FunctionName": "arn:aws:lambda:ap-northeast-1:265593583957:function:JobPollerStack-CheckLambda9CBBF9BA-Hys585A5SJk7",
      "Payload.$": "$"
    }
  }
}
```

Hãy chú ý đến thuộc tính `"End": true`, thuộc tính này chỉ ra rằng sau khi state này được chạy xong cũng sẽ là lúc StepFunction kết thúc toàn bộ hoạt động của nó.

Phía trên mới chỉ đơn thuần là định nghĩa các states riêng biệt, để thu được một sơ đồ state chuẩn chỉ như đã chỉ ra ở _Hình 1_ nêu trên, ta cần một công cụ kết nối chúng lại. Câu trả lời ở đây chính là

```ts
const definition = submitJob
  .next(waitX)
  .next(getStatus)
  .next(
    new sfn.Choice(this, "Job Complete ?")
      .when(sfn.Condition.stringEquals("$.status", "FAILED"), jobFailed)
      .when(sfn.Condition.stringEquals("$.status", "SUCCEEDED"), finalStatus)
      .otherwise(waitX)
  );
```

Bản thân code cũng đã chỉ ra rất rõ ràng rằng:

Đầu tiên là **SubmitJob**, tiếp đó là **WaitXSeconds**, sau đó là **GetStatus**.

Khi đã **GetStatus** xong, ta sẽ đi đến một điều kiện rẽ nhánh **Choice** đó là **JobComplete**, ở đây tuỳ theo giá trị của thuộc tính **status** được lấy ra từ context của stepfunction, ta sẽ đi đến:

- JobFailed
- FinalStatus
- WaitXSeconds

```ts
const stateMachine = new sfn.StateMachine(this, "CronStateMachine", {
  definition,
  timeout: cdk.Duration.minutes(5),
});
```

Cuối cùng là tạo stateMachine từ defintion được định nghĩa ở trên. Thành quả chúng ta thu được trên AWS console sẽ như sau:

[Chèn ảnh]
