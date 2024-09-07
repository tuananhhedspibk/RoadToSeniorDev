# Design And Building A Logging System

If you are a programmer, checking logs when debugging is not strange. We can not identify the file facing an error without logs or understanding how to troubleshoot.

With systems that serve a thousand of users or even a million of users in a day, checking logs is essential because:

- It can help us find the errors affecting the end users.
- Tracking the "health" of the system to "smell" some "abnormal signs" before it can outbreak to affect the system.
- ...

This shows that **checking log** is crucial when developing or operating a system, so designing and implementing a sound log system can help make monitoring easier.

I want to share my experience and understanding of designing and building a logging system in this article. I hope that through this article, you can:

1. Understand the importance of logging when operating a system.
2. Have a reference for yourself when you want to implement a real logging system.

## Logging Policy

Below is the list of the questions we should ask ourselves before implementing a logging system.

- **Why**: What is the purpose of logging?
- **Who**: What module will generate the log?
- **When**: When should we output the log?
- **Where**: Where do we output the log (send it to Slack or BigQuery, etc.)?
- **What**: What is the information that log can give?
- **How**: How to output the log?

## Log Level

Sau khi đã hiểu rõ được mục đích của mình khi đưa ra log, chúng ta cần "phân cấp" log.

After understanding the purpose of loggin we should hierarchy the log

| Log level | Concept                                                                                                                    | How to handle                               | Example                                    |
| --------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- | ------------------------------------------ |
| FATAL     | This Level hinder the operating of the system                                                                              | Have to fix immediately                     | Can not connect to the DB                  |
| ERROR     | Unexpected errors occur                                                                                                    | Should be fixed as soon as you can          | Can not send the email                     |
| WARN      | Not an error, but are some problems like unexpected input or unexpected executing unexpected input or unexpected executing | Should be refactored regularly              | Regularly delete data API                  |
| INFO      | Notification when starting or ending an executing or a transaction. Maybe outputting another needed information            | Do not need to fix                          | Output the body of the request or response |
| DEBUG     | The information that relating to system status                                                                             | Do not output in the production environment | Can be put inside a function               |
| TRACE     | Information that is more detailed than DEBUG                                                                               | Do not output in the production environment |                                            |

## Case

After splitting the log levels, we must clear `the type of the logs` we want to output.

In this section, I'll answer the following six questions for each log type.

- Why
- Who
- When
- Where
- What
- How

### System log

- Why: The system log will be used to debug when the system has errors.
- Who: The system itself will output the log.
- When: Output the log when having errors.
- Where:
  - `FATAL / ERROR`: Notify to the channel that developers can notice and handle immediately.
  - `WARN / INFO`: Output inside the system or log managing tool.
  - `DEBUG / TRACE`: Output at `console.log` inside `staging (pre-product) environment`.
- What:
  - `FATAL / ERROR`: Stacktrace.
  - `WARN / INFO / DEBUG/ TRACE`: The content we want to notify.
- How:
  - `FATAL / ERROR`: Output through log managing tools or to Slack, SMS, ... (in `push-type`).
- `WARN / INFO / DEBUG / TRACE`: Output through log managing tools or inside the system, ... (in `pull-type`).
