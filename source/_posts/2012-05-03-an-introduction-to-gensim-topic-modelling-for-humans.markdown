---
comments: true
date: 2012-05-03 14:06:02
layout: post
slug: an-introduction-to-gensim-topic-modelling-for-humans
title: 'An Introduction to gensim: "Topic Modelling for Humans"'
wordpress_id: 472
categories:
- Cool Stuff
- Interests
- Programming
tags:
- clustering
- code
- dc-python
- gensim
- lsa
- lsi
- presentation
- python
- similarity
- visularity
---

On Tuesday, I presented at the monthly DC Python meetup. My talk was an introduction to gensim, a free Python framework for topic modelling and semantic similarity using LSA/LSI and other statistical techniques. I've been using gensim on and off for several months at work, and I really appreciate its performance, clean API design, documentation, and community. (All of this is due to its creator, Radim Rehurek, who I interviewed recently.) 

The presentation slides are [available here](http://www.slideshare.net/sandinmyjoints/an-introduction-to-gensim-topic-modelling-for-humans). I also wrote some [quick gensim example code](http://williamjohnbert.com/2012/05/relatively-quick-and-easy-gensim-example-code/) that walks through creating a corpus, generating and transforming models, and using models to do semantic similarity. The code and slides are both also available on my [github account](https://github.com/sandinmyjoints/gensimtalk).

Finally, I also developed a [demo app to visualize semantic similarity queries](http://github.com/sandinmyjoints/visularity). It's a Flask web app, with gensim generating data on the backend that is clustered by scipy and scikit-learn and visualized by d3.js as agglomerative and hierarchical clusters as well as a simple table and dendrogram. To make it all work in realtime, I used threading and hookbox. I call it Visularity, and it's [available on github](http://github.com/sandinmyjoints/visularity). You need to provide your own model and dictionary data to use--check out my presentation and visit [radimrehurek.com/gensim/](http://radimrehurek.com/gensim) to learn how. Comments and feedback welcome!
