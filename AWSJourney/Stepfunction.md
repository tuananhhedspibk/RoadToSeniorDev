# Sử dụng AWS CDK để tạo Stepfunction, StateMachine

https://dev.classmethod.jp/articles/aws-step-functions-for-beginner/

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
