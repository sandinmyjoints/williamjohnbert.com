---
layout: post
title: "Allow CORS with localhost in Chrome"
date: 2013-06-25 19:04
comments: true
categories: web-development chrome programming
---

Today I spent some time wrestling with the notorious
[same origin policy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Same_origin_policy_for_JavaScript)
in order to get CORS
([cross-origin resource sharing](https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS))
working in Chrome for development work I was doing between two applications
running on localhost. Setting the `Access-Control-Allow-Origin` header to `*`
seemed to have no effect, and
[this bug report](https://code.google.com/p/chromium/issues/detail?id=67743)
nearly led me to believe that was due to a bug in Chrome that made CORS with
localhost impossible. It's not. It turned out that I also needed some other
CORs-related headers: `Access-Control-Allow-Headers` and
`Access-Control-Allow-Methods`.

This (slightly generalized) snippet of [Express.js](http://expressjs.com)
middleware is what ended up working for me:

```javascript
app.all("/api/*", function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Cache-Control, Pragma, Origin, Authorization, Content-Type, X-Requested-With");
  res.header("Access-Control-Allow-Methods", "GET, PUT, POST");
  return next();
});
```

With that, Chrome started making OPTIONS requests when I wanted to POST from
localhost:3001 to localhost:2002. It seems that using `contentType:
application/json` for POSTs forces CORS preflighting, which surprised me since
it seems like a common case for APIs, but no matter:

```javascript
app.all("/api/*", function(req, res, next) {
  if (req.method.toLowerCase() !== "options") {
    return next();
  }
  return res.send(204);
});
```
