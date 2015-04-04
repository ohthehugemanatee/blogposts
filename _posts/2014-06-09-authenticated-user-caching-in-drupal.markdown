---
layout: post
title: "Authenticated User Caching Concepts in Drupal 7"
date: 2014-06-09 22:21:01 +0200
comments: true
categories:
 - drupalplanet
 - drupal
 - performance
---
Drupal has a wide variety of highly effective solutions for caching anonymous user content. The typical setup is APC, Memcached or Redis, and Varnish in front, and this can easily serve thousands of concurrent anonymous users. There is excellent documentation out there discussing this kind of simple caching.

But what about authenticated users? You can cache elements of the page using a method like [Render cache](https://drupal.org/project/rendercache), [Entity Cache](https://drupal.org/project/entitycache), or [Views Content Cache](https://drupal.org/project/views_content_cache). But Drupal still has to assemble each page for your users, a relatively heavy operation! If you want to address hundreds or thousands of authenticated users, you're simply SOL by these traditional approaches.

Enter the [Auth Cache](https://drupal.org/project/authcache) suite of modules. Though this project has been around for quite some time, it had a reputation of being finicky and hard to set up. It got a significant rewrite in the last year thanks to [znerol](https://drupal.org/users/znerol), and is now a powerhouse of a module that brings authenticated user caching much closer to regular users.

I will say that this is still not for the squeamish. You have to really understand the building blocks of your site, and you will have to make a plan for each unique layout on your site. There are some page elements that are quite hard to build this way, but for the most part Authcache makes this easy.

The theory
---
The idea behind authenticated user caching is simple. We already have a great caching mechanism for pages that stay exactly the same for all users. So we simply identify the parts of the page that will change for each user, and use a placeholder for them instead. Think of it as a <user customized stuff here> tag in HTML. This way the page caching mechanism can ignore the customized content, and focus on the stuff that IS the same across all requests.

There are three major ways of doing this placeholder: AJAX, ESI, and Cookies.

With AJAX, you just include a little JS that says "fill this DIV with the contents of http://example.com/user/customized/thing". The client's web browser makes a second call to the server, which is configured to allow /user/customized/thing through the cache all the way to your website. Drupal (or whatever you're running) fills in the HTML that belongs in that div and returns it to the browser. Congratulations! You just served an authenticated user a page which was 99% cached. You only had to generate the contents of one div.

ESI is short for [Edge Side Includes](https://en.wikipedia.org/wiki/Edge_Side_Includes), a small extension to HTML which effectively does the same thing as that Javascript, but on the "Edge server". The Edge server is whatever service touches the HTTP request last before sending it to the client. Apache, NGINX, Varnish, pound... you want this to happen as far down the stack as you control. An ESI tag in your HTML looks like this:

```
<esi:include src="http://example.com/user/customized/thing" onerror="continue"/>
```

It's pretty clear, even to a human reader, what this tag means: "replace this tag with the contents of http://example.com/user/customized/thing". ESI actually supports some simple logic as well, but that's not really relevant to what we're doing here.

The only difference between ESI and AJAX is where the placeholder is filled. With ESI it happens on the edge service, and with AJAX it happens in the client browser. There is a subtle difference here: a page with ESI will not be delivered until all the ESI calls have returned something, while an AJAX page will return right away, even if the components don't immediately appear. On the other hand, ESI is much better for degraded browsers. YMMV.

The last method is using Cookies. You can store arbitrary information on cookies, as long as you're careful about security. That can be a very effective way to get certain limited information through a caching layer. Authcache actually comes with an example module for just such a use case. It passes the user's name and account URL in a cookie, so you can display it in a block.

This is effective for very small amounts of information, but keep it limited. Cookie headers aren't designed to hold large quantities of data, and reverse proxies can have a hard time if you put too much information in there. Still, it's a neat trick that can cover you for that "Hello Username" block.

Authcache in Drupal
---
The [Authcache](https://drupal.org/project/authcache) suite of modules tries to automatically implement AJAX and/or ESI for you. It actually goes one step further, and implements a caching layer for those "fragments" which are to be returned via ESI/AJAX. The fragments can be stored in any caching system which implements [DrupalCacheInterface](http://api.drupal.org/api/drupal/includes%21cache.inc/interface/DrupalCacheInterface/7), ie any caching module you've heard of. Memcache, APC, File Cache, Redis, MongoDB. The full page HTML with placeholders can be cached in Drupal's normal page cache, in Boost, or in Varnish.

Once you have these caching mechanisms defined, it's just a question of marking sections of your site which need a second callback. Authcache presents a large number of modules to do this. You can define any of the following as requiring a second call:

* Blocks
* Views
* Panels Panes
* Fields
* Comments
* Flags
* Forms
* Forums
* Polls
* Votes

... and that's all without writing a single line of custom code! Each one of those elements gets a new "Authcache" setting, where you can define it as needing a second callback, and set the method for the callback as either AJAX or ESI. You can even fall back to another method if the first one fails!

A good example of how this works is the Forms integration. Authcache will modify any and all forms on your site, so that they have an ESI or AJAX placeholder for the form token. This means that the form itself can be stored in your page cache (Varnish, Boost, or whatever), and the form token will be filled in automatically! That's a phenomenal speed improvement without any configuration beyond enabling the module.

Setting up Authcache is a little complicated, and I'll cover that in depth in my next post. But once the basic AJAX or ESI support is set up and these modules are enabled, caching authenticated users becomes a question of visiting each unique layout on your site and making a plan for each element that involves user customization. Authcache makes this easy.

Next post: [How to configure Authcache on Drupal 7](https://ohthehugemanatee.org/blog/2014/06/14/how-to-configure-authcache-on-drupal-7/).
