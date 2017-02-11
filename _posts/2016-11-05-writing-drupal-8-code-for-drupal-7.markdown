---
layout: post
title: "Writing Drupal 8 code for Drupal 7"
date: 2016-11-05 12:05:50 +0100
comments: true
categories: 
 - Drupal
 - Drupal 8
 - drupalplanet
 - code

---

A year ago I proposed a session for [Drupalcon Mumbai](https://events.drupal.org/asia2016) and [Drupalcon New Orleans](https://events.drupal.org/neworleans2016), called ["The best of both worlds"](https://events.drupal.org/neworleans2016/sessions/best-both-worlds-writing-drupal-8-code-drupal-7-sites). It promised to show attendees how to write Drupal 8 code for Drupal 7 sites. I never ended up giving the session, but this week I got an email asking for more information. So in case it ever comes up again, here's my own collection of resources on the subject.

The big improvement that's hard for D7 developers to get used to is injected services. The [service container module](https://www.drupal.org/project/service_container) makes that possible in D7. The brilliant [FabianX](https://www.drupal.org/u/fabianx) wrote it to make his life easier in writing [render cache](https://www.drupal.org/project/render_cache), and his is always a good example to follow! This module creates a service container for D7, which you use just like the container in D8. You can write independent, OO code that is unit testable, with service dependencies declared in a YAML file. Note that you will also need the [registry autoload](https://www.drupal.org/project/registry_autoload) module to get PS4 namespaced autoloading!

I just mentioned unit testable code as a benefit of the service container. To be honest this is a little tricksy in Drupal 7. For my own custom work I tend to isolate the test environment from the rest of Drupal, so I don't have to deal with everything else. Again, I followed Fabian's example there by looking at how [render cache does it's tests](http://cgit.drupalcode.org/render_cache/tree/tests?h=7.x-2.x). If you do want better integration, there is a good lullabot post that talks about (more) proper PHPUnit integration. https://www.lullabot.com/articles/write-unit-tests-for-your-drupal-7-code-part-1 .

Next on my list is Composer-managed dependencies. The Acquia developer blog has a great post about [using Composer Manager for this in D7](https://dev.acquia.com/blog/using-composer-manager-get-island-now). This is a huge win for a lot of custom modules, and very easy. 

Last is plugins. The rest of this list is in no particular order, but I left plugins for last because I think this isn't actually necessary in D7. Personally I use modules' own hooks and just autoload independent classes. You might consider using plugins instead if you're going to write several plugins for the same module. In any case, [Lee Rowlands has the go-to blog post about this](https://www.previousnext.com.au/blog/drupal-8-now-object-oriented-plugins-drupal-7). 

All together, you can combine these approaches to write code for D7 with the biggest Dx features of D8: service injection, phpunit testing, composer libraries, and plugins. Note that each of these blog posts assumes different workarounds for all the other functionalities... but they should help you get an understanding of how to use that particular Dx improvement in 7.

When I wrote that session proposal, I thought of this as a good way for D7 developers to learn D8 practices gradually, one at a time. I no longer think that's true. Mostly, there are so few working examples of D7 code using these practices, that it's quite hard to get your stuff working. This is particularly hard when you're just learning about the concept in the first place! Personally, I could mess around with this stuff and make my life harder with it in D7. But I couldn't really get the best advantage out of them until I had better examples. My best learning aids were the examples in D8 core, and the code scaffolding available through Drush and Drupal console. 

But now that I'm comfortable with the concepts... I would absolutely use these approaches in D7 work. You know, if I'm FORCED to work in the old system. :) 

One last aside here: it is easy to fall into the mindset that Drupal 8 practices are better just because they're newer. This is simply not true. These practices are not handed down from heaven, after all! When you have the rest of the D8 architecture in place, certain kinds of code tasks are much easier. That's why we like developing for it so much more. But other (less common, IMO) tasks are harder. And doing any of this in D7 means you have to put the architecture in place, too. That's a lot of time, and it's only worthwhile if you're going to use the particular strengths of these practices.

So if it looks like one of these D8 practices will make your life easier for a particular task in D7, then by all means use these approaches to get there. Composer manager has a particularly low bar - it's so easy to use, and makes so many tasks easier, it's a good approach to many tasks. But if I ever catch you implementing service container to get two lines of code into a form_alter, I will come to where you work and slap your hands off the keyboard. 

Happy coding!
