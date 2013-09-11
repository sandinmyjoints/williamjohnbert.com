---
layout: post
title: "How legit HTTP (with an async assist) massacred my Node workers"
date: 2013-09-10 18:55
comments: true
categories: nodejs http cluster express
---


An uncaught exception in our Node app was causing not only one, but two and then
three workers to die. (Fortunately, we hardly ever encounter uncaught
exceptions. Because we're Node studs! But still.)

The funny thing is that we're using Express, which (via Connect) wraps each
request / response in a try / catch. And we use Express's error handler, which
returns 500 on unhandled errors.

Another funny thing is we use cluster, which isolates workers from each other.
They live in entirely separate, solipsistic processes.

But instead of returning 500, our worker simply died. And, as if in sympathy,
the rest immediately followed.

Time to get to the bottom of this. A Node stud like me can figure it out, no
sweat.

For a sanity check, I went to Chrome and Firefox's network inspectors. Only one
POST, the bad request that triggered the exception. Everything else looks
normal. Sanity: verified.

Then it was on to the
[cluster module](http://nodejs.org/docs/latest/api/cluster.html#cluster_how_it_works).
That magical "OS load balancing" seemed highly suspicious. But nope, I asked in
\#nodejs and they said that only applies at the TCP connection level. Once a
connection is assigned to a worker, it never goes to another worker. Meaning
that the bad request was isolated&mdash;only the worker who received the initial
connection could encounter it.

But the workers kept on dying.

These workers morted out fast. They didn't even return 500, or any kind of
response. The more I thought about it, that didn't really seem right. Not right
at all. Why no 500?

But I can only tackle one mystery at a time. I wanted to understand: why did so
many workers die?

Furious googling ensued. My efforts were rewarded with this nugget:

> If an HTTP/1.1 client sends a request which includes a request body, but which
> does not include an Expect request-header field with the “100-continue”
> expectation, and if the client is not directly connected to an HTTP/1.1 origin
> server, and if the client sees the connection close before receiving any status
> from the server, the client SHOULD retry the request.

<small>(From the
[HTTP 1.1 spec, RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.2.4).
[Original hat tip](http://stackoverflow.com/a/14345476/599258), which links to
this
[informative post about double HTTP requests](http://geek.starbean.net/?p=393).)</small>

My mind was somewhat blown. The browers were right after all. They were just
following HTTP. And&mdash;helpfully!&mdash;hiding the resent POSTs from the network
inspector.

I had a thought. One I have often, yet each time, it seems new again: *I have
much to learn*.

Back to the 500s, or lack thereof. Which got funnier still when I realized that
other errors in our controllers that threw exceptions *did* return 500s. Being a
hands-on kind of guy, I added one right at the top of a route controller: `throw
new Error("uh-oh")`. My dev server spat back at me: `500, uh-oh.`

So why did that one particular error never, ever return a 500, or any response
of any kind?

It's my fault, really. I'm still a Node newbie (I must never forget this). I had
missed that because async IO callbacks occur in a different call stack from the
request / response cyle, one that originates from the event loop,
[Express's try / catch doesn't catch them](http://stackoverflow.com/questions/13228649/unable-to-handle-exception-with-node-js-domains-using-express/13240256#13240256).

It makes total sense. I have much to learn.

So what to do? `require('domain')` to the rescue. I can write some middleware (a
bit of [this](https://github.com/brianc/node-domain-middleware), a dash of
[that](https://github.com/mathrawka/express-domain-errors)) to wrap the request
/ response in a domain.

But how do I get this domain into my controller? My solution was to attach it to
`res.locals._domain`. Good solution? Well, I'm sure there's a better way. Good
enough? It solved my immediate problem:

```js
Model.findOne({key: value}, res.locals._domain.bind(function(err, doc) {
  // This error handler can throw all it wants. My domain will catch it.
}));
```

Sweet. Now, armed with a reference to `res` in the domain error handler, I can
return a 500. Voila, the browser gets its response. No more helpful resent
POSTs. The silent gratitude of the spared workers is its own reward.

Except, do I need to bind every
[mongoose](https://github.com/LearnBoost/mongoose/pull/1337) and
[other async IO operation](https://github.com/joyent/node/issues/3908) in my app? There
are many.

Many.

I have much to learn.
