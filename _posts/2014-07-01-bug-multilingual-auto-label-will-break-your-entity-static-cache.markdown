---
layout: post
title: "Bug: Multilingual Auto Label will break your Entity Static Cache"
date: 2014-07-01 17:00:45 +0200
comments: true
categories: 
 - drupalplanet
 - drupal
 - forumone
---
This is an important one to note: If you use the popular [Automatic Entity Label](https://www.drupal.org/project/auto_entitylabel) module on a multilingual site, [it will break your paths](https://www.drupal.org/node/2295325) because of an interaction with Drupal's built in object cache. I looked at this briefly a few months ago and ran out of time, but my (badass) colleague [bburg](https://www.drupal.org/u/bburg) figured it out this week. 

For now, the only solution is a slow one - we clear static entity caches when we generate multilingual titles. That's not an awesome fix, but it's hard to think of a better one without any of the D8 cache tagging functionality. Massive kudos to bburg for figuring this out!

And for those of you keeping score, this is a good example of how to file a bug report for a really complex issue in a really popular module... and follow up until you resolve it.
