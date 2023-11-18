# How did I build an API with DDD ? - Part 1

DDD has been and is still a popular design trend in large and highly professional systems at the present time.

However, understanding the "academic" definitions of DDD is very time-consuming and to support programmers in general as well as those who are intending to learn about DDD in particular, so the can in a short time of "absorbing" the "academic" knowledge of DDD, I would like to present to readers very basic knowledge as well as illustraive examples through this series of articles on DDD. I look forward to well received by readers.

## Overview about the API

I built this API with the main purpose to practice more about DDD so I won't focus too much on complex functions. Basically my API can be described using a usecase diagram as follows:

![Screen Shot 2023-06-01 at 8 06 37](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/941b8d88-cdcd-46b5-918f-3fa7b8d022ce)

This is basically an API that allows users to:

- Create a post with an attached photo (just like instagram)
- Comment / Like the post
- Follow / Unfollow other users

You guys can refer to my source code at: <https://github.com/tuananhhedspibk/NewAnigram-BE-DDD>

In this API, the two main technologies I used are:

- `nestjs` framework
- `mysql` database

## What is DDD ?

DDD (Domain Driven Design) is a way of designing "business-oriented" software - that is, taking the business or domain logic as the center of the system.

An example can be the system "banking operations" or "logistics operations".

If we only talk about concepts, it will be difficult to imagine what DDD is, so in this article I will try to illustrate each concept of DDD corresponding to my API code.

### Basic concepts in DDD

Because it is a "domain" driven design, **domain** is the main concept here. Revolving around the concept of domain, we have 3 main key words: **Domain**, **Model** and **Domain Model**.

**Domain** is a certain business field such as "banking business" or "logistics business", ...

**Model** is the process of abstracting objects as well as business operations. This process of modeling business into abstract objects is called **Modeling** or **Modeling**.

**Domain Model** are the models we obtain after the **Modeling** process above.

Going back to the API above, my APIs domain is `users have the ability to post and follow other users`. I will model it into 2 main models:

- User
- Post

These are also the two main "objects" in my API. To illustrate the above two models more clearly, I have the Domain Model diagram as follows:

![Screen Shot 2023-06-01 at 8 02 43](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/e438f205-c319-4211-b607-edb7f163105c)

Looking through this diagram, you can see:

- Basic properties of post: content, imageUrls, ...
- Basic user attributes: email, userName, password
- 1-n relationship between User-Post.

It's correct but still missing a lot, for example:

- Comments
- Like
- Following
- User detail (save user details such as: firstName, lastName, age, ...)

Not only that, in DDD we also have another very important concept which is **Aggregate**. Simply explained, an aggregate is a set of objects that are closely related to each other in terms of data and we must always ensure that.

For example:

![Screen Shot 2023-06-01 at 8 16 16](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/43e4e501-01aa-46c9-94d7-33bd9d0b19c8)

A Club will have "Members", here we stipulate that the Club's status will be `FULL` if the value of `memberCount = 5`, the Club's status will be `STILL_FREE` if the value of `memberCount < 5`. From this rule, we see that between Club and Member there is a fairly close data relationship, so we can conclude that Club and Member will belong to the same aggregate. So the domain model diagram of Club and Member will be revised as follows:

![Screen Shot 2023-06-01 at 8 20 34](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/45fd9793-169b-45fa-9a6b-58c4d47b8c9f)

Going back to the API, after considering Follow, Like, Comment and User Detail, I decided to divide the aggregate as follows:

![Screen Shot 2023-06-01 at 21 29 19](https://github.com/tuananhhedspibk/DataIntensiveApp/assets/15076665/3a0741f2-c6af-4b2d-add6-52e2bdfcda09)

A brief explanation is as follows: I divided my domain into 3 aggregates:

- User
- Post
- Follow

My API doesn't have as many data constraints as the Club and Member examples above, but organizing it in aggregate "units" will help the code express the business more clearly.

Take the example of User Aggregate: in this aggregate I store the user's basic information (email, password) and detailed information (nickName, avatarUrl) of the user through 2 objects `User` and `UserDetail` respectively. , here `User` will be called `Root Aggregate`.

One of the basic principles when working with aggregates is that aggregate information can only be updated through the `root aggregate`, meaning that here, updating information for `UserDetail` or `User` must be done through `User Aggregate` or in other words `User Object`.

You can see how I define `User Aggreate` at [this address](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/blob/main/src/domain/entity/user/index.ts) . You can see how I update `UserDetail` as follows:

```ts
export class UserEntity extends BaseEntity {
  updateDetail(params: UpdateDetailParams) {
    if (!this.detail) {
      this.detail = new UserDetailEntity();
    }

    if (params.avatarURL) {
      this.detail.avatarURL = params.avatarURL;
    }

    if (params.gender) {
      this.detail.gender = params.gender;
    }

    if (params.nickName !== null) {
      this.detail.nickName = params.nickName;
    }
  }
}
```

I created an `updateDetail` method inside the `UserEntity` class (this is the User Aggregate class), this method will update the user detail information so the actual user detail update will look like this:

```ts
const user = new UserEntity(); // Define User Aggregate

user.updateDetail({nickName: "testUser"}); // Update detail through user aggregate
```

And User aggregate will "aggregate" userDetail within it through the class property as follows:

```ts
class UserEntity extends BaseEntity {
  detail: UserDetailEntity;
}

class UserDetailEntity extends BaseEntity {
  id?: number;
  active: boolean;
  nickName: string;
  avatarURL: string;
  gender: UserDetailGender;

  constructor() {
    super();
  }
}
```

Each method defined in the Aggregate class itself will represent the content of the business that the system is modeling, in the example above it is `updateDetail`.

Similar with `Post Aggregate` at [source code](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/blob/main/src/domain/entity/post/index.ts). `Post Aggregate` with `Post object` as the `root aggregate`, `Comment` & `Like` objects will be child objects that are "aggregated" inside `Post Aggregate` through class properties as follows:

```ts
class PostEntity extends BaseEntity {
  likes: LikeEntity[];
  comments: CommentEntity[];
}

class CommentEntity extends BaseEntity {
  id?: number;
  content: string;
  userId: number;
  postId: number;
}

class LikeEntity extends BaseEntity {
  id?: number;
  userId: number;
  postId: number;
}
```

As for `Follow Aggregate`, it is quite simple, it only has 2 unique properties:

- srcUserId: id of the user is following.
- destUserId: id of the user being followed.

More specifically, you can read [here](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD/blob/main/src/domain/entity/follow.ts)

## Wrap up

OK, so that's it, a very basic and important concept of DDD is **Aggregate**. Hopefully, through this introductory article, you have absorbed the first few concepts of DDD. Basically, after this section, I hope you can understand:

- What is a domain?
- What is Modeling?
- What is Domain Model?
- What is Aggregate? What characteristics does Aggregate have that are worth paying attention to?

There is also a small reminder for you that the two diagrams I presented above are `Usecase diagram` and `Domain model diagram`, I will not present in detail the definitions of the two types but its could be explained briefly as follow:

- `Usecase diagram` will show the main functions that the system will provide to users (this diagram must come **before** the domain model diagram)
- `Domain model diagram` will show how we model objects in our system. In this diagram, you need to pay attention to the following basic things:

① Divide the aggregate appropriately based on actual operations.

② Specify the properties of the aggregate as well as the child objects in it (you may not need to write the method).

③ Fully explain the business logic in the diagram (in bullet form).

This is the end of Part 1. See you again in the next articles in the series about DDD, thank you.
