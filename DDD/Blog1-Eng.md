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
