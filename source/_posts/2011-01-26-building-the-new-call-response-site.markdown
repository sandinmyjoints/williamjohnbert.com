---
comments: true
date: 2011-01-26 20:32:03
layout: post
slug: building-the-new-call-response-site
title: Building the New Call + Response Site
wordpress_id: 184
categories:
- Interests
- Projects
- Web Development
tags:
- call + response
- jquery
- sartorii
- web development
- wordpress
---

[![National Portrart Gallery in the snow](http://williamjohnbert.com/wp-content/uploads/2011/01/5391220127_9ed0503fb0_b.jpg)](http://williamjohnbert.com/wp-content/uploads/2011/01/5391220127_9ed0503fb0_b.jpg)Over the past couple weeks I worked on relaunching the website for [Call + Response](http://www.callandresponsedc.org/), an art show I co-curate with my friend [Kira](http://twitter.com/kiraface).

The website for the second iteration of the show, Call + Response: Textures, is pretty different from [the first version](http://williamjohnbert.com/callandresponse2010/). The first was ultra minimal, hand-coded, and didn't use a CMS—just a simple site that I could get going in a matter of hours. It was not designed to coordinate with a promotional campaign incorporating Twitter and Facebook. Even doing regular updates on our progress putting together the show would have been a real pain.

My goal for this site was to keep the clean, minimal feel while using a modern CMS that would allow for frequent updates, integration with social media, and cool stuff like photo galleries, maps, and interactive widgets. So here's what I did.



	
  * First I installed  a Wordpress theme, a rather minimalist one called [sartorii](http://www.yukei.net/proyectos/satori-english-documentation/). Then I started to customize it to the particular needs of the site, slimming it down further, eliminating sidebars and meta information, culling whatever felt extraneous to the site's mission of communicating essentials and focusing on the concept and participants.

	
  * To present the bios of the participants, I used jQuery UI's accordion tool. Call + Response pairs artists with writers, so I wrote code to link two accordions together. When you click on an artist name you get two bios, theirs and the writer they are paired with, and vice versa.

	
  * I installed  a lightbox plugin for the [photos page](http://www.callandresponsedc.org/?page_id=65). First, I discovered just how many lightbox plugins are out there. I wanted one that would integrate with Wordpress's gallery feature, and it took some time to find a solution. I ended up using [Fancy Gallery](http://wordpress.org/extend/plugins/fancy-gallery/), which integrates with [Fancy Box](http://fancybox.net/) to produce the desired results.

	
  * Once I received the poster design for the show from our designer, [Oliver Munday](http://www.olivermunday.com), I further customized the theme to use the poster's colors and imagery.


I'm pleased with the result. I think it fulfills the goals I set forth. I learned a ton about jQuery, css, and WordPress, which was a great side benefit. Picking up new skills can be really enjoyable, especially when  those skills quickly turn into tangible results.

I'm already starting on my  next web development project: building a Django app to manage a list of DC-area reading series.
