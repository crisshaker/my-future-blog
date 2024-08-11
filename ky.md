Version: 1.1

# Let’s talk about ky.js

This is a 2-part (or maybe 3-part) article about ky.js. In this first part, we'll talk about what `ky` is and why you should consider using it. In the second part, we'll explore some of the features that make `ky` a great library to work with. If there's a third part, we'll explore some advanced features and/or use cases for `ky`.

### How I got here

If you're working on any sort of web project, it's not uncommon to run into a situation where you need to make an HTTP request. Different projects run in different environments, have different requirements, and as such, you might need to make requests in different ways.

In the past, you might have used `XMLHttpRequest`, at least, that's what I started with. Working with `XMLHttpRequest` directly was nothing short of a nightmare. It was a lot of boilerplate, and it was not fun to work with. I mean, it worked, but it was not fun.

But the JS ecosystem being what it is, third-party libraries were created to make working with `XMLHttpRequest` a lot easier. Many of such libraries were created, and the one that I used the most was axios. Axios was easy to use and worked well. It was significantly easier and more pleasant to work with than `XMLHttpRequest`.

JavaScript itself is constantly evolving, and the web platform is evolving with it. Part of this evolution was the introduction of the fetch API. Fetch was a breath of fresh air. It was a lot easier, and more pleasant to work with than `XMLHttpRequest`. And unlike `XMLHttpRequest`, you typically don't need a third-party library to enjoy working with fetch. It is great out of the box. I love fetch, and I hope we at least have that in common 😄.

I have a confession to make though. Even long after fetch was introduced, I still used axios in my projects because it "just worked." Seriously, I had no reason to switch to fetch. I mean, why would I? Axios was working just fine for me. Well, it worked fine until it didn't.

Now, it's important to note that the problem was not with axios itself. The problem was that axios was built on top of `XMLHttpRequest`, at least, when running in the browser. I was working on a chrome extension, and I found out that service workers in Manifest version 3 extensions don't have support for `XMLHttpRequest`. I had to switch to fetch, and I was not happy about it. I mean, I love fetch, but I was so used to axios that I didn't want to switch.

The switch was primarily hard because not only did axios make working with `XMLHttpRequest` easier, but it also had a lot of features that I didn't even know I needed. Really, I was spoiled by axios. From customizable instances to interceptors, axios was a great library. Switching to fetch meant that I wasn't going lose all of these features.

Naturally, I had 2 options. I could either write my own wrapper around fetch or find a library that did what I needed. I chose the latter, and that's how I found ky. To some extent, you could say `ky` is to fetch what axios is to `XMLHttpRequest`.

I've come to like `ky` so much that even in new projects where I could and would have used axios, I now use ky. I decided to write these articles to share my experience with `ky` and why I think you should consider using it.

### What is ky?

ky is a tiny (no dependencies), elegant HTTP client. It's built on top of the fetch API so naturally, it should work in any environment that supports fetch.

### Important defaults

In this article, I'm going to argue that `ky` is a drop-in replacement for fetch with extra features when you need them. To make the statement entirely true, I should mention that `ky` comes with 2 important default behaviors that you should be aware of.

#### 1. Behavior with non-2xx status codes

If you’ve worked with fetch, you most probably know that fetch **only** throws an error if the request fails due to a network error or if anything prevents the request from completing. This is the default behavior of fetch and `ky` works exactly the same way, but with a slight difference.

In addition to the default behavior of fetch, `ky` also treats non-2xx status codes as errors. This means `ky` will throw an error if the response status code is not in the range of `200-299`. If you've worked with axios, you might be familiar with this behavior.

In some cases, this behavior is desirable. In fact, fetch or otherwise, libraries like tanstack-query (sort of) expect this kind of behavior to make it easier to handle errors. However, if you want to override this behavior, you can simply pass in the `throwHttpErrors` option as `false`.

```js
import ky from 'ky';

const response = ky(url, {
  throwHttpErrors: false,
});
```

#### 2. Request retries

By default, `ky` will retry failed requests 2 times. In most cases, this behavior might not even be noticeable. However, if you want to override this behavior, you can simply pass in the `retry` option with the number of retries you want, in this case, `0`.

```js
import ky from 'ky';

const response = ky(url, {
  retry: 0,
});
```

Having to do this for every request can be cumbersome. If you want to set this once for all requests, you can create an instance of `ky` with these options.

```js
import ky from 'ky';

const superFetch = ky.create({
  throwHttpErrors: false,
  retry: 0,
});

const response = superFetch(url);

// You can still override the defaults per request if you want
const response = superFetch(url, {
  retry: 1,
});
```

With this, you can now make requests with `ky` and have it behave exactly how you want it to. In fact, you can replace every fetch call in your project with `ky` and have it work exactly the same way. The following examples are equivalent.

```js
// Using fetch
fetch(url)
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));

// Using ky
superFetch(url)
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

```js
const request = new Request(url);

// Using fetch
fetch(request)
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));

// Using ky
superFetch(request)
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

```js
// Using fetch
fetch(url, {
  method: 'POST',
  body: JSON.stringify({
    title: 'foo',
    body: 'bar',
    userId: 1,
  }),
  headers: {
    'Content-Type': 'application/json',
  },
})
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));

// Using ky
superFetch(url, {
  method: 'POST',
  body: JSON.stringify({
    title: 'foo',
    body: 'bar',
    userId: 1,
  }),
  headers: {
    'Content-Type': 'application/json',
  },
})
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

In case you haven't figured it out yet, the point of these examples is that `ky` is a drop-in replacement for fetch with **extra features** when you need them. Really, `ky` is fetch! And if you’re already using fetch in your project, you can easily switch to `ky` without any issues.

Now, what do I mean by extra features? Let's start with something simple. The previous examples can be simplified with ky.

```js
// using fetch
fetch(url, {
  method: 'POST',
  body: JSON.stringify({
    title: 'foo',
    body: 'bar',
    userId: 1,
  }),
  headers: {
    'Content-Type': 'application/json',
  },
})
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));

// using ky
superFetch
  .post(url, {
    json: {
      title: 'foo',
      body: 'bar',
      userId: 1,
    },
  })
  .json()
  .then((data) => console.log(data));
```

You're probably saying to yourself, "That's not much of a difference. Besides, I can easily create a helper function to do that." And you're right! Honestly, if this was all `ky` had to offer, I wouldn't be writing this article. But `ky` has a lot more to offer.

In the next part of this article, we'll explore some of the features that make `ky` a great library to work with.
