---
layout: post
title: "HTTP 1.1 killed all my Node workers"
date: 2013-09-08 15:54
comments: true
categories:
---



(Coming soon: a tldr; how to handle node errors summary post.)

This is what I want to happen when our Node app encounters an unhandled error:

* The request that triggered the error gets a 500 response.
* The worker that encountered the exception shuts down, gracefully. (We're using
  the cluster module.)
* A new worker is spun up to replace the dead one.

(Already I'm simplifying: what if the error isn't triggered inside a request /
response cycle? Will deal with that later.)

(And should the new worker be spun up when the old one actually dies, or when it
stops accepting new connections? These are the questions that haunt me now.)

Notice that I used the word "gracefully". What does gracefully mean? It means
that the worker process:

1. Stops accepting new connections.
2. Cleanly closes out any other connections or in-flight requests if
possible, though if the error is with, say, the db connection pool that might
not be possible.
3. Sets a hard timer on how long it will try to cleanly close existing connections.
4. Dies when all connections are closed or the timer expires, whichever comes
  first.

I've got about a year of node under my belt. Much of it was as the junior
developer on a team of two. I was sheltered from the nuts and bolts of some of
this stuff. The senior dev was the sole dev before I started, and he ported a
very successfull (~6M uniques/month) site to use a Node backend, complete with
Chef deploys, very cool stuff that impressed me when I started.

We never seemed to have any trouble with our deploys. So I figured this was all
covered.

I hadn't really looked at the exception handling parts of our app. I just knew
that a) it hardly threw any uncaught errors, so that's good, b) usually when it
did, for example during dev, it returned 500 and printed a stack trace, so that
seemed ok, but c) sometimes in production we had seen a bunch of workers die
instead of just one.

Was our app doing this? Nope. Was it doing anything like this? Nope!

There are uncaught exceptions, and unhandled errors. Which were which? Unhandled
errors did not crash the app. They returned 500. Uncaught exceptions did crash
it. If I just threw inside a controller, it was an unhandled error and I got
a 500. But this one example bad request would generate an uncaught exception.
What was the difference?

It stems from two different sources (sources?) of errors: Express error
middleware, and `process.on("uncaughtException")`.

On an uncaught exception (which are very rare, but we did have one case that
triggered them) the worker went down. Hard. The opposite of gracefully.

And then another worker followed it like a lemming off a cliff. And then
another. This was pretty much the opposite of what I wanted. About the only thing
that was working right was that the cluster master was spinning up new workers
to replace the dead.

How was this possible?

Where were our workers killing themselves? There was no `process.exit` in our
source. In our logging library, a custom lib designed to marry Winston and
Express, in the uncaught exception handler.

Step 1 was to replace this. The logger does not get to `exit(1)` the process.
Instead, with a reference to the `server` returned by Express's
`app.listen`, on uncaught the app would now do `server.close` and only exit
inside its callback.

Node's `server.close` is graceful.

`server.close` might not always succeed in closing out its connections. So I
added a `setTimeout.`

OK, so now we're at least shutting down somewhat gracefully. I went back to
trigger the exception.

A worker died.

Then another.

Then another.

Why so many needless deaths?

It wasn't be the client. There was only one POST in Chrome's network inspector.
For good measure, I tried Firefox, too. Same result.

At this point, I wondered how the master handed out requests to its workers.
Node docs said "OS load balancing". They all share a handle to a socket. So
could the OS be noticing that the process it just gave the request to died, and
therefore handing it to the next process?

That was my chief suspicion. I asked around on \#nodejs. Nope, that's not -- the
OS and socks work on the level of connections (duh), not requests, and once a
connection is assigned to a worker, it stays with that worker. So the same
request from the same client could not be handed off to another worker.

But that was the behavior I saw again and again. Always 3 workers got the same
request. Always they died. I changed my number of workers from 4 to 2. Same
behavior. When worker 1 died, worker 3 came up. Worker 2 died, then worker 3.

I logged the POST, figuring I'd only see one if it was node, cluster master, or
the OS passing the bad request/connection around. But there were three POSTs.
The client **was sending three POSTs**. That went against everything I knew
about POSTs. They have side effects! They can't be repeated.

Then I found this: http://stackoverflow.com/questions/14302512

HTTP 1.1 says you *should* resend requests with bodies (e.g., POSTs) when the
connection is closed. Chrome and Firefox were doing what the spec said. (Though
their network inspectors were a little misleading.)

Naively, I had been thinking it was ok to have one connection that gets no
response if can clean up the others ok. Nope. According to http, on connection
close you should retry requests with bodies such as posts. So in comes the same
request that just brought down your worker! And boom another one goes down.
Chrome will try a total of 3 times.

This brought me back to uncaught exceptions vs unhandled errors. Why were
unhandled errors handled ok, but uncaught exceptions caused the worker to die?

Because of Express. Express wraps each request / response cycle in a try /
catch. When it catches, we handled that as an unhandled error, and used
Express's error handler to return 500.

But what was the difference? How could I trigger an uncaught exception? My
example one was thrown **in a Mongoose query callback**. Therefore it's in a
different call stack from the wrapped request / response cycle:
http://stackoverflow.com/a/13240256/599258  It's triggered from the event loop.
Nothing is catching it.

But I needed to catch it in order to close out the bad connection so that
spec-compliant user agents wouldn't repeat their POSTs and kill my workers.

Enter the domain.

Node provides domains.

There are some packages out there for wrapping Express request / response cycles
in domains. I looked at them and wrote middleware of my own that combined the
best of what I saw. That was the easy part.

The hard part was how to use the domain in my async IO callback. I ended up
adding the domain to `res.locals` and then binding it to the IO callback
manually:

```js
Model.findOne({id: params.id}, res.locals._domain.bind(function(err, result) {
  // Error handler.
}));
```

My domain had an `on "error"` listener that now got triggered when the callback
threw. So the uncaught exception became an unhandled error, went through the
usual Express error handling middleware, and returned 500.

Awesome.

Chrome did not resend its POSTs. Only one worker died. And it died gracefully.
The others didn't even notice, just went on their business.

But we have async IO callbacks throughout the server. Do I need to manually bind
them to the domain everywhere? This is an open question for me. I have seen
presentations on domains and read blog posts.
[This looks interesting](http://stackoverflow.com/a/14304061/599258). But I
still don't have an answer.

The domain error handler has a reference to `res` so it can return 500
gracefully. That's why an application-level server domain doens't work: without
a ref to the `res`, I can't return 500, so browsers will resend their POSTs!


Anyway, I'm off to Hawaii for two weeks for my honeymoon. See ya.



# Notes

Error handling and shutting down / restarting are messy in Node.

Express tries to help but makes it particularly messy.

Close isn't graceful enough. (Why not?)

We want a new worker to start up as soon as a bad one stops accepting
connections, not waiting until it actually dies. Is this true? Then we might
have n+1 workers on n CPUs. But one shouldn't be doing much.

We want workers to gracefully exit on a signal we send, not just on an uncaught
exception.

Service stop and service start aren't going to cut it. Need to send signal to
master process. Master has to know what to do with it.

Still don't know what is good solution for catching on async callbacks. Maybe we
have architected wrong.

Combination of: Handling uncaught from event loop callback. truly graceful
exiting. Good monitoring. Really understanding what is happening under the hood.
Nice abstractions get blown away!
