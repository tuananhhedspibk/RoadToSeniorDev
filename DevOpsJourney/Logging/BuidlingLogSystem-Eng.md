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

### Access log

- Why: Ouptut the log to follow the sending and receiving request process.
- Who: System itself or from infrastructure.
- When: Output at sending or receiving request time.
- Where: In `level INFO` and `pull-type.` Because the amount of logs may be large, we have to pay attention to finding log speed.
- What: Output **who**, **how**, **when** access to the system.
- How: Depending on the purpose, there may be some differences.

### Action log

- Why: Output to analyze user actions; based on that, we can improve the service.
- Who: System itself or external tools.
- When: Some actions occur.
- Where: Tools that can analyze logs (BigQuery, ...).
- What: Depending on the purpose.
- How: Depending on the purpose, there may be some differences.

### Auth log

- Why: Output to tracking user authentication.
- Who: System itself.
- When: Authenticate user.
- Where: In `level INFO` and `pull-type`
- What: Output **who**, **how**, **when** is authenticated.
- How: Depending on the authenticate method, there may be some differences.

## Example

That's all about the concepts; let's look at a sample project.

You can see here for more details about the code: <https://github.com/tuananhhedspibk/NewAnigram-BE-DDD>

### Selecting loggin library

I'll use `log4js` library (<https://github.com/log4js-node/log4js-node>), the reason is simple because `log4js` constructs their logging level in the same way with my thinking.

### Implementing

#### Step 1 - Define a logger class

First, I'll define a logger class like this:

```ts
class Logger {
  public default: log4js.Logger;
  public system: log4js.Logger;
  public api: log4js.Logger;
  public access_req: log4js.Logger;
  public access_res: log4js.Logger;
  public sql: log4js.Logger;
  public auth: log4js.Logger;

  public fatal: log4js.Logger;
  public error: log4js.Logger;
  public warn: log4js.Logger;
  public info: log4js.Logger;
  public debug: log4js.Logger;
  public trace: log4js.Logger;

  constructor() {
    log4js.configure(loggerConfig);

    this.system = log4js.getLogger('system');
    this.api = log4js.getLogger('api');
    this.access_req = log4js.getLogger('access_req');
    this.access_res = log4js.getLogger('access_res');
    this.sql = log4js.getLogger('sql');
    this.auth = log4js.getLogger('auth');

    this.fatal = log4js.getLogger('fatal');
    this.fatal.level = log4js.levels.FATAL;

    this.error = log4js.getLogger('error');
    this.error.level = log4js.levels.ERROR;

    this.warn = log4js.getLogger('warn');
    this.warn.level = log4js.levels.WARN;

    this.info = log4js.getLogger('info');
    this.info.level = log4js.levels.INFO;

    this.debug = log4js.getLogger('debug');
    this.debug.level = log4js.levels.DEBUG;

    this.trace = log4js.getLogger('trace');
    this.trace.level = log4js.levels.TRACE;
  }
}
```

I defined `Log Levels` inside of class Logger:

- fatal
- error
- warn
- info
- debug
- trace

beside of that, I defined `Log types`:

- system
- api
- access_req
- access_res
- sql
- auth

#### Step 2 - Apply Logger into project

I'll apply the Logger class into project that is implemented by NestJS framework.

You can see [here](https://docs.nestjs.com/) for more details about NestJS framework.

By using `Interceptor` feature (<https://docs.nestjs.com/interceptors>) of NestJS, I'll "inject" my logger into project through Interceptor.

The reason that makes me choose "Interceptor" is NestJS Interceptor can wrap not only `request stream` but also `response stream` comes to and go out from API so using Interceptor is the easiet way to catch `request log` and `response log`. I defined `LoggerInterceptor` class like this:

```ts
export class LoggerInterceptor implements NestInterceptor {
  intercept(
    context: ExecutionContext,
    next: CallHandler<any>
  ): Observable<any> | Promise<Observable<any>> {
    // intercept() method will "wrap" request/ response stream

    /*
     * Get request object from context
     * After that, pass request object to "requestLogger" function
     * to output the log
     */
    const request = context.switchToHttp().getRequest();
    requestLogger(request);

    /*
     * Get response object from context
     * After that pass response object to "responseLogger" & "responseErrorLogger" functions for ouputting the log or
     * error log
     */
    const response = context.switchToHttp().getResponse();

    return next.handle().pipe(
      // 200 - Success Response
      map((data) => {
        responseLogger({ requestId: request._id, response, data });
      }),
      // 4xx, 5xx - Error Response
      tap(null, (exception: HttpException | Error) => {
        try {
          responseErrorLogger({ requestId: request._id, exception });
        } catch (e) {
          logger.access_res.error(e);
        }
      })
    );
  }
}
```

you can read [here](https://github.com/tuananhhedspibk/restful-app/blob/main/libs/interceptor/logger/index.ts) for more details.
