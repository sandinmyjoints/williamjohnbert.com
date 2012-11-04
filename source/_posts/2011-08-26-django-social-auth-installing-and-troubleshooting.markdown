---
comments: true
date: 2011-08-26 11:16:14
layout: post
slug: django-social-auth-installing-and-troubleshooting
title: 'django-social-auth: Installing and troubleshooting'
wordpress_id: 374
categories:
- Cool Stuff
- Projects
- Web Development
---

Thanks to `django-registration`, I was able to build a working account registration/login system pretty easily. But I wanted to give users the ability to use their existing accounts through popular services such as Facebook, Twitter, etc., rather than have to create yet another account. Here's how I did it.

# Sorting Through the Choices #

There are a number of reusable Django apps out there to help with registration/login from social media sites. I found this [Review of 4 Django Social Auth apps](http://hackerluddite.wordpress.com/2011/05/17/review-of-4-django-social-auth-apps/) very helpful in sorting out the options. After reading it, I was left to choose between [`django-social-auth`](https://github.com/omab/django-social-auth) (I originally linked to the wrong app here, but this link is correct) and [`django-allauth`](https://github.com/pennersr/django-allauth). In the end, I went with `django-social-auth` (not to be confused with `django-socialauth`) because a friend had recommended it and because I'd already installed it before I read this article. However, the article's conclusion that `django-allauth` is best out of the box also seems valid.

# Installation #

The instructions in [`django-social-auth`'s docs](http://django-social-auth.readthedocs.org/en/latest/) are helpful in walking you through available settings and options. 

I also found the included example app useful. To use this app, I cloned `django-social-auth`'s git repo, created a virtualenv called `django-social-auth`, ran `pip install -r requirements.txt` inside this virtualenv to install all the required apps, ran `manage.py syncdb`, and finally ran `manage.py runserver`. Voila, example app is up and running at 127.0.0.1, showing a simple screen with options to login through about a dozen different different services.

# API Keys #

The first service I tested was Twitter. I use it more than any others, and I already had the API keys for it. I threw my API key and secret key into the example `local_settings.py` file provided with `django-social-auth` and tried to log in via the example app. Boom: `401 Unauthorized`. I double-checked all my settings and installation and whatnot. Seemed fine. 

I turned my attention to the API keys. The ones I had were generated for [Readsr](http://www.readsrs.com), i.e., I entered readsrs.com as the domain when I generated them at dev.twitter.com. But now I was running on localhost, 127.0.0.1, so I suspected the readsrs.com keys wouldn't be valid. I wasn't sure whether Twitter would hand over a new consumer key for 127.0.0.1, or baulk at the request. (It seemed like it should do so, but I hadn't seen any instructions anywhere that said to get a key for your development machine.) Turns out Twitter will happily give you a key for 127.0.0.1. Once I plugged the new keys in, I was able to log in with my Twitter credentials, and just as it should, `django-social-auth` automatically created an `auth.user` for this account. 

# Integrating with Readsr #

I followed the instructions again to config my own app, Readsr. To add a login option using Twitter credentials, I put a link to the reversed view that begins the `django-social-auth` login process for twitter, i.e., `{% url socialauth_begin "twitter" %}`, to my login template. And it worked.

I still need to fix a few oddities. For example, Twitter returns my first and last names together in `first_name` (or else `django-social-auth` is concatenating them into that column), and doesn't supply any email address. But the basic functionality is there, and was relatively easy to achieve.

# Postscript #

The author of the article I linked above had an error using OpenID when using `django-social-auth`, which is why he preferred `django-authall`. He filed a bug for the error he got, and I notice that [it was closed](https://github.com/omab/django-social-auth/issues/67) 15 hours ago (though if you read the comments, it seems it was actually fixed back in mid-July). Good timing.
