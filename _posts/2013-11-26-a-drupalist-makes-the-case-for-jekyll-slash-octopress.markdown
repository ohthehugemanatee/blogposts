---
layout: post
title: "A Drupalist makes the case for Jekyll/Octopress"
date: 2013-11-26 17:59:05 -0500
comments: true
categories: 
 - Jekyll/Octopress
---

Today I'm launching my new domain, [OhTheHugeManatee.org](https://ohthehugemanatee.org). ORG was the only TLD left, but I'm asking after .com . :)

I made the decision to launch it using [Jekyll](http://jekyllrb.com/ "Jekyll") and [Octopress](http://octopress.org "Octopress"), rather than Drupal. This feels a little funny to a career Drupalist like myself. Why not use Drupal 7, or use it as a testing ground for Drupal 8? Anyone who knows me will know that I complain bitterly about Ruby, and I've been vocal about the misuse of Jekyll for [healthcare.gov](http://healthcare.gov "Healthcare.gov") . So I feel that an explanation is due, and probably appropriate for my first post written on the system.

First of all, *I'm a strong believer in the best tool for the job*. I believe that Drupal is the best tool for a lot of complex content management tasks, and I'm excited to work with Drupal 8 as a contributor and implementer. But a single user blog is not a complex content management task. Maybe at some point I will start up a multi-faceted website that covers everything I do, from code to opera to ninjutsu... but that's not today, and it's not the objective for this site. For this site, I want a tool that makes it trivially easy to create, edit, and publish blog posts, using an editing interface that I'm comfortable with. Octopress is a great fit for that requirement.

Second, though I love my work with Drupal, *I'm not blind to the rest of the world*. For the rest of the tech world, Jekyll is a leading blog platform. As I've specialized more and more in Drupal, I feel the limitations of "Drinking the Kool-Ade" more and more. Drupal is ascendant right now, but it won't always be. Someday I'll have to move onto another framework, and getting comfortable with a different, but still popular, content framework can only be helpful. Better still that it's written in a language I'm not comfortable with.

Third, *it's genuinely a great platform*. There's a lot to learn from success, and Octopress is definitely having success among my peers. I'm looking forward to playing with this new toy, and hopefully bringing some new insights into my home platform.

The migration process is not difficult, as long as you know which scripts to use. Outside of the Drupal world there's a lot of confusion between D6 and D7, and it took me awhile to figure out that the importer I was running was for D6. For those who come after me, I used [this post](http://approache.com/blog/migrating-from-blogger-to-octopress/) and it's links to migrate the content from my old blogger site, [swearing at computers](http://swearingatcomputers.blogspot.com). Then I used [Jekyll's own doco](http://import.jekyllrb.com/docs/drupal7/) to get my posts from the [5 Rings blog](http://5ringsweb.com/blog). I'm self hosted because I don't like plaintext transmission, and Github/Heroku don't provide free HTTPS hosting. Besides, I've got all these servers lying around, I may as well use them. :)  After an abortive attempt to deploy myself using a simple git repo for the HTML, I ended up doing it the Jekyll way with rsync and "rake deploy". Pretty slick.
