---
layout:     post
title:      Design your own HTTP Framework
subtitle:   Delving into HTTP
date:       2023-08-02
author:     Minyi
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - HTTP
    - Distribute System
    - Golang
---

# Concept of HTTP

The Hypertext Transfer Protocol (HTTP) is an application layer protocol in the Internet protocol suite model for distributed, collaborative, hypermedia information systems. (wiki)

## Methods

The methods GET, HEAD, and POST are defined as cacheable. In contrast, the methods PUT, DELETE, CONNECT, OPTIONS, TRACE, and PATCH are not cacheable.

## Status Code

`1XX` (informational): The request was received, continuing process.

`2XX` (successful): The request was successfully received, understood, and accepted.

`3XX` (redirection): Further action needs to be taken in order to complete the request.

`4XX` (client error): The request contains bad syntax or cannot be fulfilled.

`5XX` (server error): The server failed to fulfill an apparently valid request.

# Http framework design

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F17c9efed-59f1-4ef7-955d-855107784526_2180x992.png)

Http framework with aforementioned design will be more clear.

## Middleware Design

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0e579197-a1f4-41f4-8d73-fd05efde52a3_1302x792.png)


A good example is KOA:

We register middleware using the `app.use` method. Multiple middlewares can be registered, and their execution order depends on the order of registration; the ones registered earlier are executed first.

Middleware is simply a function that accepts two parameters provided by Koa:

1.  `ctx`: The context object.
1.  `next`: The next function.

The `ctx` object contains various parameters, such as the request object (`request`) and the response object (`response`).

Calling the `next` function will execute the next middleware in the chain. If you don't call the `next` function, the next middleware will not be executed.

```
const Koa = require('koa');

const app = new Koa();

// Middleware 1
app.use(async (ctx, next) => {
  console.log('Middleware 1');
  await next();
  console.log('Complete');
})

// Middleware 2
app.use(async (ctx, next) => {
  console.log('Middleware 2')
  const data = await getData(); 
  ctx.body = { data };
})

// Simulate getting data from a database
const getData = async () => {
  return 'Hello World!';
}

app.listen(3000);
```

1.  Execute the code before `next()`.
1.  Then, execute all the code of the middleware 2 that comes after `next()`.
1.  Finally, execute the code after `next()`.

Therefore, the log looks like this:

```
Middleware 1
Middleware 2
Complete
```

#### Follow up:

If user doesn’t call next manually, we still can set a limit for index and stop it:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9913c679-3775-413d-9ec3-a7df856667dc_1174x398.png)

## Route Design

Using a prefix tree (also known as a trie) can make querying more convenient. ( You can find out some questions in leetcode about tire. LOL.)

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1f434006-442d-4dc7-8adc-2a17a9133b3e_1476x998.png)

Data structure will be like:

｜ map | method | trie | node |

Meanwhile, you can use a list to record handlers chain

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fab4b9feb-1a99-4191-9aaf-81a99c7f18c3_1150x520.png)

## Codec Design

Contexts should not be stored inside a struct type, but instead passed to each function that needs it. This article expands on that advice with reasons and examples describing why it's important to pass Context rather than store it in another type: <https://go.dev/blog/context-and-structs>

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbcd09576-6c12-49b2-9ee7-5e4b25bedafc_1204x194.png)

## Transport Design

BIO uses a one-to-one mapping between threads and connections, while NIO uses a one-to-many mapping, allowing for more efficient handling of I/O operations with fewer threads.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd9195987-f3dc-41a5-ae0c-f2be291a4d3f_2242x988.png)

BIO: go net, manage buffer by users

NIO: netpoll is a good example - <https://github.com/cloudwego/netpoll>

## How to optimize

-   Header decode uses SIMD

-   Request context pool

    ![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F91af9243-4cc5-4960-ac80-86eb247e1749_2030x906.png)

-   Buffer design

> https://open.substack.com/pub/minyizou/p/coming-soon?r=2k1xwy&utm_campaign=post&utm_medium=web
