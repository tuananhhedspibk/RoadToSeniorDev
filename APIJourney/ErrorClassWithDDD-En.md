# Build error-handling classes for DDD and Clean Architecture

## The need to have error-handling classes

Besides ensuring that the business logic is executed correctly, we must ensure that our code covers as many errors as possible when building software or web applications. Let's talk about the error. Usually, we have two types of errors:

- Errors that occur by user actions (4xx error)
- Errors that occur by framework or our "bad logic code" (5xx error)

Our ultimate aim is to create software that can serve users anytime, anyplace, and provide them with the best experiences. To achieve this, we must handle the two types of errors that can occur, ensuring that our users' experience is always our top priority.

In this article, I will show you how it designed and built error-handling classes for a DDD project. Let's dive in.

## Build error-handling classes for DDD and Clean Architecture

### Design

Let's talk about DDD and Clean Architecture. Unlike traditional architecture like MVC or MVVM, DDD & Clean Architecture make the business logic the center of the entire application. We will separate our application into layers, each with its mission. Here, we will have four layers:

1. Domain layer: this is the heart of the entire application; it includes the central business logic.
2. Use-case layer: this layer includes services and features.
3. Presentation layer: this layer is the same as the controller layer of MVC architecture
4. Infrastructure layer: this outage layer has the responsibility to contact external API or database

And pay attention that the inside layer can not be dependent on the outer layer - we call this the `Dependecies Rules`

![291058405-83885e17-766b-4a95-9c4e-b26d52182215](https://github.com/user-attachments/assets/63e4ce72-a399-434b-b7c8-c192533778b4)

### Coding

### Executing
