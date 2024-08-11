# Let’s talk about ky.js

This is a 2-part or maybe 3-part article about ky.js. In this first part, we'll talk about what ky is and why you should consider using it. In the second part, we'll explore some of the features that make ky a great library to work with. If there's a third part, we'll explore some advanced features of ky.

### A bit of my personal history

This is a story about how I found ky and fell in love with it. Please feel free to skip to the next section if you couldn't care less about my personal history 😅

If you're working on any sort of web project, it's not uncommon to run into a situation where you need to make an HTTP request. Different projects run in different environments, have different requirements, and as such, you might need to make requests in different ways.

In the past, you might have used XMLHttpRequest, at least, that's what I started with. Working with XMLHttpRequest directly was nothing short of a nightmare. Thankfully, libraries like axios made working with XMLHttpRequest a lot easier.

Eventually, the fetch API was introduced, and it was a breath of fresh air. Fetch made working with HTTP requests a lot easier and more pleasant. It was a huge improvement over XMLHttpRequest, and it was a lot easier to work with. I love fetch, and I hope we at least have that in common 😄.

I have a confession to make though. Even long after fetch was introduced, I still used axios in my projects because it "just worked." Seriously, I had no reason to switch to fetch. I mean, why would I? Axios was working just fine for me. Well, it worked fine until it didn't.

Now, the problem was not with axios itself. The problem was that axios was built on top of XMLHttpRequest, and I was working on a chrome extension (manifest v3) and it didn't support XMLHttpRequest. I had to switch to fetch, and I was not happy about it. I mean, I love fetch, but I was so used to axios that I didn't want to switch.

I was spoiled by axios. From customizable instances to interceptors, axios had everything I needed. Naturally, I had 2 options. I could either write my own wrapper around fetch or find a library that did what I needed. I chose the latter, and that's how I found ky. To some extent, you could say ky is to fetch what axios is to XMLHttpRequest.

I love ky so much that even in new projects where I could and would have used axios, I now use ky. I love it so much that I decided to write this article to share my love for ky with you. I hope you find it as useful as I have.

### What is ky?

ky is a tiny (no dependencies), elegant HTTP client. It's built on top of the fetch API so naturally, it should work in any environment that supports fetch.

### Important defaults

ky comes with a few important defaults that you should be aware of.

#### 1. Behavior with non-2xx status codes

If you’ve worked with fetch, you most probably know that fetch **only** throws an error if the request fails due to a network error or if anything prevents the request from completing. ky works exactly the same way, but with a slight difference.

In addition to the default behavior of fetch, by default, ky also treats non-2xx status codes as errors. This means ky will also throw an error if the response status code is not in the range of `200-299`. If you've worked with axios, you might be familiar with this behavior.

In some cases, this behavior is desirable. In fact, fetch or otherwise, libraries like tanstack-query (sort of) expect this kind of behavior to make it easier to handle errors. However, if you want to override this behavior, you can simply pass in the `throwHttpErrors` option as `false`.

```js
import ky from 'ky';

const response = ky(url, {
  throwHttpErrors: false,
});
```

#### 2. Request retries

By default, ky will retry failed requests 2 times. In most cases, this behavior might not even be noticeable. However, if you want to override this behavior, you can simply pass in the `retry` option with the number of retries you want, in this case, `0`.

```js
import ky from 'ky';

const response = ky(url, {
  retry: 0,
});
```

Having to do this for every request can be cumbersome. If you want to set this once for all requests, you can create an instance of ky with these options.

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

With this, you can now make requests with ky and have it behave exactly how you want it to. In fact, you can replace every fetch call in your project with ky and have it work exactly the same way. The following examples are equivalent.

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

You might be wondering what the whole point of these examples is. Well, the point is that ky is a drop-in replacement for fetch with **extra features** when you need them. Really, ky is fetch! And if you’re already using fetch in your project, you can easily switch to ky without any issues.

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

You're probably saying to yourself, "That's not much of a difference. Besides, I can easily create a helper function to do that." And you're right! Honestly, if this was all ky had to offer, I wouldn't be writing this article. But ky has a lot more to offer.

In the next part of this article, we'll explore some of the features that make ky a great library to work with.
