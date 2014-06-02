---
layout: post
title: "Using Github Pages to hand off a legacy site and make everyone happier"
date: 2014-04-30 18:52
comments: true
categories: git github legacy django
---

Here's how I turned over maintenance of a legacy site -- built as a one-off
project years ago using now outdated technology -- to my non-techincal
cofounder, with only a few hours of work. Best of all, it now uses evergreen
technology that will make it easy for her to update for years to come, and
everyone is happy with the outcome.

### The problem

I created [Call and Response](http://www.callandresponsedc.org) when my friend
Kira and I decided to co-curate an art show of that name in 2010. It became an
annual event, and each year I updated the website with the new participants and
details. This year, Kira is continuing the show with support from others because
I'm out of capacity to help.

Kira needs to be be able to update the website. It lists what the concept behind
the show is; who's participating; where the show is; when it opens; and more. I
created the original website using [Django](http://www.djangoproject.com) as a
learning exercise when I was fairly new to web development. I went all out by my
standards of the time, creating a simple micro-CMS that helped me learn about
the MVC pattern, etc.

However, the project had no use outside of this particular website. It could not
be generalized without a significant overhaul, and I had no need to generalize
it. In fact, I had no interest in maintaining it or even working with it. I
built it with Django 1.3, and hosted it with a service that I don't use anymore.

Usually, in such a case, it's tough luck for the the web developer. Kira is not
a web developer and has no interest in becoming one; she just wants to be able
to update her website. The responsible thing is to step up to to the task, and
keep the site going.

This time I had an idea for how to reach the end goal of the site being updated
in a timely way yet still ditching the legacy code. I thought I saw a path
forward to handing it off to her in a way that would let her easily edit text,
links, and images, and create new pages.

### The solution

Here's how I converted the site from a Django project that only someone who knew
Python could update into a flat site that anyone could edit by knowing only the
most basic HTML:

\1. Scrape.

  This seemed like something I could do fairly easily with my current web
  programming tool of choice -- Node.js. I just had too download and write to disk
  every page I could find by following links that had the same host. But often
  with things that seem straightforward, the devil's often in the details, and
  anyway, I figured someone else must have wanted to do this, so there had to be a
  tool for it, which would turn this from fairly easily into dead simple.

  The tool I found is called [HTTrack](http://www.httrack.com/).

    $ brew install httrack

  I ran it against [Call and Response](http://www.callandresponsedc.org),
  selecting all the default options.

    $ httrack "www.callandresponsedc.org" -v

  Lo and behold, it did exactly what I wanted. Now I had a full static version of
  the site.

\2. Host.

I needed to host this static site somewhere free, reliable, and easy to use. I
created a Github organization, initialized a git repo, created a branch named
`gh-pages`, and pushed it to Github. Now the site was up at
`callandresponsedc.github.io/callandresponsedc`, (albeit in a broken form due to
the static assets' root-relative paths resulting in 404s).

Then I added a
[CNAME](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages)
for callandresponsedc.org, and I switched the DNS from my old hosting provider
to the name service, and added the
[right DNS entries](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap/)

Once the DNS changes took effect, the new static site was up where it needed to
be: http://callandresponsedc.org. And it looked *exactly* as it had before: to
any visitor who saw it before the change and again after, no difference would
have been apparent.

\3. Enable.

While I'd waited for the DNS to change, I'd asked Kira to create a github account,
added her to the organization, and spent 15 minutes writing up a brief guide to
how to edit the HTML pages using Github.

Thanks to
[github's editing tools](https://github.com/blog/905-edit-like-an-ace), she now
had the ability to make changes that would take effect instantly.

Technically, at this point, I was no longer needed. I could have handed it over
now. Kira was enabled to do everything she needed to do. But I would have had to
add a caveat: "By the way, when you want to change anything in the menu, or
header, or footer, or to add a new logo or change colors, you have to do it on
*every single page*."

Because Httrack, good as it is, is not good enough to produce DRY results. Each
page of the static site repeated all the same HTML structure for its header and
footer and body, and contained its own set of inline styles. Changing any part
of that would have meant changing every page. There weren't a lot pages, but
nonetheless, I didn't want to force that burden on K.

I could do better.

\4. Refactor.

Large parts of the pages were identical, so I could factor them out into
includes, and make a couple of simple layout templates composed of these. Same
idea with the stylesheet.

To make this work, I had to use something slightly more complicated than flat
HTML files. I decided to convert the site to use
[(Github-flavored) Jekyll](https://help.github.com/articles/using-jekyll-with-pages).
I won't repeat the instructions needed to get that up and running; just follow
the link, and do what it says. It takes on the order of a few minutes.

To refactor, I just needed to figure out exactly what was common to all pages,
hack it out into includes, and reassemble the includes as templates. Then I'd
specify which template to use via YAML front-matter on each page, and voila, my
Jekyll site would be done. Because I use
[Emacs](http://www.gnu.org/software/emacs/), I used
[ediff](http://www.gnu.org/software/emacs/manual/html_mono/ediff.html), but you
could do this with any diffing tool and text editor.

A nice side effect of this was that the pages that Kira would be editing got much
simple. They went from something like:

```html
<!DOCTYPE HTML>
<html lang="en">
  <meta http-equiv="content-type" content="text/html;charset=utf-8" />
<head>
  <title>About - Call + Response
  </title>
  <script src="http://platform.twitter.com/anywhere.js?id=IrVoVLmkJDVw9Pagwdxsow&amp;v=1"
          type="text/javascript">
  </script>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Call + Response is an art show in Washington, DC, pairing writers and artists.">
  <meta name="author" content="William John Bert">

  <link rel="stylesheet" type="text/css" href="/static/fonts/fonts.css">
  <link rel="stylesheet" type="text/css" href="/static/css/bootstrap.css"/>
  <link rel="stylesheet" type="text/css" href="/static/css/bootstrap-responsive.css"/>
  <link rel="stylesheet" type="text/css" href="/static/css/style.css"/>

</head>

<div id="fb-root"></div>
<script>(function(d, s, id) {
    var js, fjs = d.getElementsByTagName(s)[0];
    if (d.getElementById(id)) return;
    js = d.createElement(s); js.id = id;
    js.src = "http://connect.facebook.net/en_US/all.js#xfbml=1&appId=398123750217250";
    fjs.parentNode.insertBefore(js, fjs);
}(document, 'script', 'facebook-jssdk'));</script>


  <body>
    <div class="navbar navbar-fixed-top">
  <div class="navbar-inner">
    <div class="container-fluid">
      <a class="brand" href="/">Call + Response
      </a>
    </div>
  </div>
</div>

    <div class="container-fluid">
      <div class="row-fluid">
        <div class="span3">
  <div class="well sidebar-nav">
    <ul class="nav nav-list">
      <li class="">
        <a href="/about">About
        </a>
      </li>
      <li class="">
        <a href="/opening">Opening
        </a>
      </li>
      <li class="">
        <a href="/participants">Participants
        </a>
      </li>
      <li class="">
        <a href="/sponsors">Sponsors
        </a>
      </li>
      <li class="">
        <a href="/contact">Contact
        </a>
      </li>
      <li class="">
        <a href="/press">Press
        </a>
      </li>
    </ul>
  </div>
</div>

        <div class="span9">
          <div class="chiclet pull-right visible-desktop">
  <a href="https://twitter.com/share" class="twitter-share-button" data-count="none"
     data-hashtags="callandresponsedc">Tweet
  </a>
  <script>!function (d, s, id) {
    var js, fjs = d.getElementsByTagName(s)[0];
    if (!d.getElementById(id)) {
    js = d.createElement(s);
    js.id = id;
    js.src = "http://platform.twitter.com/widgets.js";
    fjs.parentNode.insertBefore(js, fjs);
    }
    }(document, "script", "twitter-wjs");
  </script>
</div>
<div class="chiclet pull-right visible-desktop">
  <div class="fb-like" data-send="true" data-width="100"
       data-show-faces="false" data-colorscheme=light>
  </div>
</div>

          <div class="swappable-content">
            <h1>About
</h1>
<p>
  Call + Response is an (almost) annual art show in the nation's capital that brings together writers and visual artists. The writers provide the call with an original piece of writing and then visual artists generate a new piece of work in response. The end result being two pieces that resonate with each other.
  [etc...]
```

to:

```html
---
layout: page
title: About
---

<h1>About
</h1>
<p>
  Call + Response is an (almost) annual art show in the nation's capital that brings together writers and visual artists. The writers provide the call with an original piece of writing and then visual artists generate a new piece of work in response. The end result being two pieces that resonate with each other.
</p>
```

It took an hour or so to DRY out everything, and then...I was done. A site that
had been a mess -- one-off code written years ago for an outdated version of a
framework that I don't use anymore -- had become so simple that I could hand it
off to my non-technical cofounder and both of us would be happier for it!

And sure enough, since then, Kira has made numerous updates to the site, each of
which would have taken at least one, maybe a couple, rounds of emails back and
forth, plus a context switch for me to make the changes, and a deploy, another
round of emails to confirm that everything now looked good. Repeat, repeat,
repeat.

I did this with a Django site, but it'd be just as applicable with a lot of
Wordpress sites, or any CMS, really. Obviously, there are many cases where this
solution wouldn't work for a variety of reasons. But if you can make it work,
it's a joy to do so!
