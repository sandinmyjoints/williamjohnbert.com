---
comments: true
date: 2012-04-30 12:58:42
layout: post
slug: interview-with-radim-rehurek-creator-of-gensim
title: Interview with Radim Rehurek, creator of gensim
wordpress_id: 465
categories:
- Cool Stuff
- Interests
- Programming
tags:
- gensim
- lda
- lsi
- python
- semantic-similarity
- similarity
- topic-modeling
---

Tomorrow at the [May 2012 DC Python meetup](http://meetup.dcpython.org/events/23832731/), I'm giving a talk on [gensim](http://radimrehurek.com/gensim/), a Python framework for topic modeling that I use at work and on my own for semantic similarity comparisons. (I'll post the slides and example code for the talk soon.) I've found gensim to be a useful and well-designed tool, and pretty much all credit for it goes to its creator, Radim Rehurek. Radim was kind enough to answer a few questions I sent him about gensim's history and goals, and about his background and interests.

**WB: Why did you create gensim?**

RR: Consulting gig for a digital library project (Czech Digital
Mathematics Library, dml.cz), some 3 years ago. It started off as a
few loosely connected Python scripts to support the "show similar
articles" functionality. We wanted to use some of the statistical
methods, like latent semantic analysis. Originally, gensim only
contained wrappers around existing Fortran libraries for SVD, like
Propack and Svdpack.

But there were issues with that, and it scaled badly (all documents in
RAM), so I started looking for more scalable, online algorithms.
Running these popular methods shouldn't be so hard, I thought!

In the end, I developed new algorithms for these methods for gensim.
The theoretical part of this research later turned into a part of my
PhD thesis.


**Who is using gensim (as far as you know)--academics, hobbyists, commercial entities, a mixture? Any particularly interesting uses?**

Yes, I've heard from many academic as well as commercial
organizations, both on the mailing list and off. Off the top of my
head: ravn.co.uk, roistr.com, sportsauthority.com, larkc.eu; TU of
Denmark, U of Stuttgart, Masaryk U, U of Ghent, some people used it in
the Yahoo! KD cup competition... But what they all did with gensim, or
whether they still use it, I don't know. The gensim license (LGPL) is
pretty liberal in that respect.

Unfortunately, all this use rarely translates into any feedback or
contributions. I guess I'm just not very good at the
bring-new-developers-and-grow-open-source stuff :(


**Roughly how much of the current codebase was written by you, and how much by contributors?**

Almost everything by me, but I am very grateful for bug fixes and
patches. I try to put every contribution from other people into the
changelog: https://github.com/piskvorky/gensim/blob/develop/CHANGELOG.txt
. I made some wiki pages to make contributing easier:
https://github.com/piskvorky/gensim/wiki . I also try to answer
general questions on the mailing list.


**What are your favorite features, or parts of the code that you're most proud of?**

I don't have emotional attachments to parts of the code -- if it's
bad, it needs to go. I guess the most proven parts are the ones that
had been around for the longest -- LSA etc. Things that were
contributed recently by other people, like the new HDP (hierarchical
dirichlet process) code, or the `gensim.parsing` subpackage, are the
most rough around the edges.

The best feature is the memory independence for sure. Most
implementations of the statistical semantics methods assume the
training data resides in RAM, which limits their use to small/medium
corpora. Also they work in batch mode, needing a full re-train when
new training data arrives. The LSA/LDA algos are online though (can be
updated with new data, incrementally).


**What's your background? Academic, software engineering, both?**

I finished my PhD, but I feel more like a software engineer than a
pure researcher. Even during my academic years, I was working in IT
commerce. I wouldn't like to stay in academia professionally.


**What are you working on next for gensim? What about outside of gensim?**

Small things like adding the "hashing trick" etc:
https://github.com/piskvorky/gensim/issues . Basically things that
gensim users have been asking for. Some issues keep coming back on the
mailing list, and while not technically bugs, they hint at minor
redesigns and improvements.

One big thing that is missing is a basic visual style for gensim. I
have no clue how to do that and it's really pathetic gensim doesn't
even have a logo yet!

Outside of gensim, I am busy doing consulting (scaling up text
processing: fulltext search, semantic search, ad targeting etc --
backend stuff). I'm planning to do a startup that offers semantic
search and similarity as a service. A kind of easy-to-use black box
tool, something like searchify or myrrix. But it's hard to find good
people to work with... and hard to give up/interrupt a well-paying
career :) I applied for YC last month, alone, but they turned me down.
