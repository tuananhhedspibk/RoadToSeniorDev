# Build error-handling classes for DDD and Clean Architecture with Typescript

## The need to have error-handling classes

Besides ensuring that the business logic is executed correctly, we must ensure that our code covers as many errors as possible when building software or web applications. Let's talk about the error. Usually, we have two types of errors:

- Errors that occur by user actions (4xx error)
- Errors that occur by framework or our "bad logic code" (5xx error)

Our ultimate aim is to create software that can serve users anytime, anyplace, and provide them with the best experiences. To achieve this, we must handle the two types of errors that can occur, ensuring that our users' experience is always our top priority.

In this article, I will show you how it designed and built error-handling classes for a DDD project. Let's dive in.

## Build error-handling classes for DDD and Clean Architecture

### Design and coding

#### A litte bit about DDD

Let's talk about DDD and Clean Architecture. Unlike traditional architecture like MVC or MVVM, DDD & Clean Architecture make the business logic the center of the entire application. We will separate our application into layers, each with its mission. Here, we will have four layers:

1. Domain layer: this is the heart of the entire application; it includes the central business logic.
2. Use-case layer: this layer includes services and features.
3. Presentation layer: this layer is the same as the controller layer of MVC architecture
4. Infrastructure layer: this outage layer has the responsibility to contact external API or database

And pay attention that the inside layer can not be dependent on the outer layer - we call this the `Dependecies Rules`

![291058405-83885e17-766b-4a95-9c4e-b26d52182215](https://github.com/user-attachments/assets/63e4ce72-a399-434b-b7c8-c192533778b4)

In the above picture, three red arrows illustrate the `Dependencies Rules`: The use-case layer must depend on the domain layer, and the Presentation layer must depend on the Use-case layer.

#### Base classes

OK, that is enough for DDD. Now, we will move on to our main topic **Error-Handling Classes**. I separate my application into four layers:

- Domain layer
- Use-case layer
- Presentation layer
- Infrastructure layer

As a result of this division, I've created four error classes, each corresponding to one of the four layers. This is to ensure that we have a structured approach to handling errors across the application.

```ts
abstract class BaseError extends Error {}

class DomainError extends BaseError {
  // Omitted ...
}

class UsecaseError extends BaseError {
  // Omitted ...
}

class PresentationError extends BaseError {
  // Omitted ...
}

class InfrastructureError extends BaseError {
  // Omitted ...
}
```

Let's break down the above code snippet. First, I have defined the base class `BaseError`—it is just an abstract class that extends from the `Error` interface of Typescript.

Like I said above

> I will define four error classes correspond to four layers

I have defined four error class:

- `DomainError` for the Domain layer.
- `UsecaseError` for the Use-case layer.
- `PresentationError` for the Presentation layer.
- `InfrastructureError` for the Infrastructure layer.

When checking log or reading errors, we usually care about the following:

- Error code
- Error message

So I've decided to define two properties, `code` and `message,` for each of the error classes, but the error classes I've created above are extended from the `Error` interface of Typescript.

```ts
interface Error {
  name: string;
  message: string;
  stack?: string;
}
```

Because my four classes have a `message` property, I don't need to define it inside the class. But I want my error object to provide more information than only `code` and `message,` so I decided to add the `info` property to each class like this.

```ts
class DomainError extends BaseError {
  code: string;
  info?: { [key: string]: any };
}
```

Because I want the info property to store various information types, I have defined it in a free style—just an object with a string key and a value that can be any data type.

#### Error code

Now is `Error Code`. I want to tell you the property `code` of the above error classes.

Well, we can use the type `string`, but I want to have something fixed, something more structured, so I've decided to define `Error Enum`. With typescript, you can define `enum` like this:

```ts
enum ErrorCode {
  InternalServerError,
}
```

However, have some reasons for not choosing `Enum`. Instead of that, I am going to define a constant like this:

```ts
const ErrorCode = {
  SOME_ERROR: 'SOME_ERROR',
} as const;
```

The main reason is that I want to have an immutable literal error code. Another article will explain the difference between `as const` and `enum` in more detail.

Back to error classes, I am going to define error code constants like this:

```ts
const DomainErrorCode = {
  BAD_REQUEST: 'BAD_REQUEST',
  NOT_FOUND: 'NOT_FOUND',
  INTERNAL_SERVER_ERROR: 'INTERNAL_SERVER_ERROR',
} as const;

type DomainErrorCode = (typeof DomainErrorCode)[keyof typeof DomainErrorCode];
```

By doing this I'll have a type `DomainErrorCode` like this:

```ts
// 'BAD_REQUEST' | 'NOT_FOUND' | 'INTERNAL_SERVER_ERROR'
```

I will do the same thing with the `UsecaseErrorCode`, `PresentationErrorCode`, and `InfrastructureErrorCode` data types which correspond to the `Usecase layer`, `Presentation layer`, and `Infrastructure layer`.

Now I want to show you one more thing: `ErrorDetailCode`. You can ask: 'What is this?' simply it's just an

> Error code that gives you more detail about where the error is occurring or what business logic or what API is getting an error

The reason I want to define it is that `ErrorCode` is just a common code for `Bad Request Error—400`, `Not Found Error—404`, or `Internal Server Error—500`. With `ErrorCode` only, we can not know exactly where the error is occurring, what API is involved, or what business logic is facing the error.

I'll do the same thing to define the `ErrorDetailCode` type as I did with `ErrorCode,` for example, for the domain layer.

```ts
const DomainErrorDetailCode = {
  INVALID_PASSWORD: 'INVALID_PASSWORD',
  INVALID_USERNAME: 'INVALID_USERNAME',
};

type DomainErrorDetailCode =
  (typeof DomainErrorDetailCode)[keyof typeof DomainErrorDetailCode];
```

And of course, I'll have the same thing for `UsecaseErrorDetailCode`, `PresentationErrorDetailCode`, `InfrastructureErrorDetailCode`.

Finally, we will have the entire ErrorCode class like this:

```ts
const DomainErrorDetailCode = {
  INVALID_PASSWORD: 'INVALID_PASSWORD',
  INVALID_USERNAME: 'INVALID_USERNAME',
};

type DomainErrorDetailCode =
  (typeof DomainErrorDetailCode)[keyof typeof DomainErrorDetailCode];

const DomainErrorCode = {
  BAD_REQUEST: 'BAD_REQUEST',
  NOT_FOUND: 'NOT_FOUND',
  INTERNAL_SERVER_ERROR: 'INTERNAL_SERVER_ERROR',
} as const;

type DomainErrorCode = (typeof DomainErrorCode)[keyof typeof DomainErrorCode];

interface DomainErrorParams {
  info?: { [key: string]: unknown };
  code: DomainErrorCode; // This DomainErrorCode class is the thing that I wanted
  message: string;
}

export class DomainError extends Error {
  info?: { [key: string]: unknown };
  code: DomainErrorCode;
  message: string;

  constructor(params: DomainErrorParams) {
    super();

    this.code = params.code;
    this.message = params.message;
    this.info = params.info || null;
  }
}
```

OK, that is something about designing and some snippet codes about the `ErrorCode` class. In the next section, I'll show you how to use it.

### Applying

Let's take an example of a web application's signup feature. I've used the DDD for my web application so the signup logic will go to the use-case layer.

You can see more details here.

<https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/blob/main/src/usecase/authentication/signup/index.ts#L78>

I'll take a code snippet to explain.

```ts
async execute(command: SignupUsecaseInput) {
  const { email, password } = input;

  if (!email || !password) {
    throw new UsecaseError({
      message: 'Must specify email and password',
      code: UsecaseErrorCode.BAD_REQUEST,
      info: {
        detailCode: UsecaseErrorDetailCode.MUST_SPECIFY_EMAIL_AND_PASSWORD,
      },
    });
  }
}
```

You can see that after receiving the email and password parameters, I will check their lengths; if one of those is equal to 0, I'm going to throw a `UsecaseError` to notice that the email or password has a problem, like this:

In the above picture, all of this information:

- Error Type
- Error Message
- Error Code
- Error Detail Code

is shown.

Just throwing an error with an error message is enough. Exactly, I felt the same thing before, but it has some cons as the following:

- The `Error Message` can be too long for the client's screen size, so the user can not read the entire message; instead of using `Error Message`, the client can use `Error Code` or `Error Detail Code` to map with the `Error Message` that fits the user's screen size.
- `Error Code` and `Error Detail Code` make our error more specific, making our debugging process easier.

That is, you can think that defining an error class system is too "engineering" and does not provide any pros to the business. OK, you are not wrong, but if you want your system to become maintainable and easy to recover, I highly recommend defining your system's own error classes.

Thanks for reading; I'll see you in the next blog. Happy coding =)))
