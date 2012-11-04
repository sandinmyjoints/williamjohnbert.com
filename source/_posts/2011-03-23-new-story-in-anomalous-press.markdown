---
comments: true
date: 2011-03-23 11:35:36
layout: post
slug: new-story-in-anomalous-press
title: New story, and other updates
wordpress_id: 230
categories:
- Publications
- Web Development
- Writing
tags:
- anomalous press
- django deployment
- new story
- pay with a tweet
---

[![Anomalous Press #1](http://williamjohnbert.com/wp-content/uploads/2011/03/anomalous1.png)](http://williamjohnbert.com/wp-content/uploads/2011/03/anomalous1.png)I'm really thrilled to have a story, ["Winner,"](http://www.anomalouspress.org/current/21.bert.winner.php) in the very first issue of Anomalous Press. It was released on the Ides of March, so I'm a bit tardy announcing it here—it's been a busy March so far.

The issue includes work from many better writers than me, including [a poem](http://www.anomalouspress.org/current/25.ayala.winter.php) and [two translations](http://www.anomalouspress.org/current/26.ayala.vision.php) from my friend Naomi Ayala. It was Naomi who suggested I submit to Anomalous. In this way and many others, she's been a great friend and supporter of my writing, and I hope someday to be able to pay her back. You can catch another poem of hers when [Call + Response: Textures](http://www.callandresponsedc.org/) opens on April 16. (More on that tomorrow.)

Anomalous Press #1 is available for download as a PDF, but only if you [pay with a tweet](http://www.paywithatweet.com/). Neat concept. Attention as currency. And from a technical standpoint, I'm interested in how they integrated with [Twitter using OAuth](http://dev.twitter.com/)—I want to do a similar thing with the reading series web app (Readsr/Readthing/whatever) I'm working on: tweet whenever a reading series is updated. So I'll be looking into this soon.

First, though, I plan to deploy an alpha version of the site this weekend. I've been reading up on [virtualenv](http://pypi.python.org/pypi/virtualenv), [pip](http://pypi.python.org/pypi/pip), [fabric](http://fabfile.org/), and [git](http://git-scm.com/), trying to figure out how to deploy the right way--with automated version control and dependency isolation. This part, the admin stuff, is as complicated as any coding problem I've faced. Looking forward to tackling it in a marathon session on Saturday!
