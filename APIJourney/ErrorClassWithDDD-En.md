# Build error-handling classes for DDD and Clean Architecture

## The need to have error-handling classes

Besides ensuring that the business logic is executed correctly, we must ensure that our code covers as many errors as possible when building software or web applications. Let's talk about the error. Usually, we have two types of errors:

- Errors that occur by user actions (4xx error)
- Errors that occur by framework or our "bad logic code" (5xx error)

Our ultimate aim is to create software that can serve users anytime, anyplace, and provide them with the best experiences. To achieve this, we must handle the two types of errors that can occur, ensuring that our users' experience is always our top priority.

In this article, I will show you how it designed and built error-handling classes for a DDD project. Let's dive in.

## Build error-handling classes for DDD and Clean Architecture

### Design and coding

Let's talk about DDD and Clean Architecture. Unlike traditional architecture like MVC or MVVM, DDD & Clean Architecture make the business logic the center of the entire application. We will separate our application into layers, each with its mission. Here, we will have four layers:

1. Domain layer: this is the heart of the entire application; it includes the central business logic.
2. Use-case layer: this layer includes services and features.
3. Presentation layer: this layer is the same as the controller layer of MVC architecture
4. Infrastructure layer: this outage layer has the responsibility to contact external API or database

And pay attention that the inside layer can not be dependent on the outer layer - we call this the `Dependecies Rules`

![291058405-83885e17-766b-4a95-9c4e-b26d52182215](https://github.com/user-attachments/assets/63e4ce72-a399-434b-b7c8-c192533778b4)

In the above picture, three red arrows illustrate the `Dependencies Rules`: The use-case layer must depend on the domain layer, and the Presentation layer must depend on the Use-case layer.

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

### Applying
