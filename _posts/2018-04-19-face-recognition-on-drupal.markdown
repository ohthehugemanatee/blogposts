---
layout: post
title: "Drupal Does Face Recognition: Introducing Image Auto Tag module"
date: 2018-04-19 18:06:51 +0200
comments: true
categories: 
  - drupal
  - drupalcon
  - machine learning
  - microsoft
---

Last week I wrote a Drupal module that uses face recognition to automatically tag images with the people in them. You can find it on [Github](https://github.com/ohthehugemanatee/image_auto_tag), of course. With this module, you can add an image to a node, and automatically populate an entity_reference field with the names of the people in the image. This isn't such a big deal for individual nodes of course; it's really interesting for bulk use cases, like Digital Asset Management systems.

{% img center /images/image-auto-tag.gif Automatic tags, now in a Gif. %}

I had a great time at Drupalcon Nashville, reconnecting with friends, mentors, and colleagues as always. But this time I had some fresh perspective. After 3 months working with Microsoft's (badass) CSE unit - building cutting edge proofs-of-concept for some of their biggest customers - the contrast was powerful. The Drupal core development team are famously obsessive about code quality and about optimizing the experience for developers and users. The velocity in the platform is truly amazing. But we're missing out on a lot of the recent stuff that large organizations are building in their more custom applications. You may have noticed the same: all the cool kids are posting about Machine Learning, sentiment analysis, and computer vision. We don't see any of that at Drupalcon.

There's no reason to miss out on this stuff, though. Services like Azure are making it extremely easy to do all of these things, layering simple HTTP-based APIs on top of the complexity. As far as I can tell, the biggest obstacle is that there aren't well defined standards for how to interact with these kinds of services, so it's hard to make a generic module for them. This isn't like the Lucene/Solr/ElasticSearch world, where one set of syntax - indeed, one model of how to think of content and communicate with a search-specialized service - has come to dominate. Great modules like search_api depend on these conceptual similarities between backends, and they just don't exist yet for cognitive services.

So I set out to try and explore those problems in a Drupal module.

**Image Auto Tag** is my first experiment. It works, and I encourage you to play around with it, but please don't even think of using it in production yet. It's a starting point for how we might build an analog to the great [search_api](https://drupal.org/project/search_api) framework, for cognitive services rather than search.

I built it on Azure's Cognitive Services Face API to start. Since the service is free for up to 5000 requests per month, this seemed like a place that most Drupalists would feel comfortable playing. Next up I'll abstract the Azure portion of it into a plugin system, and try to define a common interface that makes sense whether it's referring to Azure cognitive services, or a self-hosted, open source system like [OpenFace](https://cmusatyalab.github.io/openface/). That's the actual "hard work".

In the meantime, I'll continue to make this more robust with more tests, an easier UI, asynchronous operations, and so on. At a minimum it'll become a solid "Azure Face Detection" module for Drupal, but I would love to make it more generally useful than that.

Comments, Issues, and helpful PRs are welcome.
