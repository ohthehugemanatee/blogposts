---
layout: post
title: "Drupal 8 RC 1 is out! What now?"
date: 2015-10-08 11:16:02 +0200
comments: true
categories: 
- Drupal
- drupalplanet
- Drupal 8
---
Last night (my time) I got the good news over twitter:

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Blue smoke from the chimney, I repeat blue smoke from the chimney!&#10;&#10;<a href="https://twitter.com/hashtag/Drupal8?src=hash">#Drupal8</a> release candidate 1 has been released! Good day for <a href="https://twitter.com/hashtag/Drupal?src=hash">#Drupal</a>!</p>&mdash; Marc Drummond (@MarcDrummond) <a href="https://twitter.com/MarcDrummond/status/651870155828412416">October 7, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

That's right, Drupal 8 has it's first release. But what does that mean? Is it done? Can I start using it yet? What kind of changes are coming? Will dawehner get to sleep, at last?

Are we there yet?
---

Despite all the rejoicing on social media, this isn't the final release for Drupal 8 - it's only the first Release Candidate. This means that we have (officially!) 0 "critical" bugs left to fix in Drupal 8. That means exactly what it sounds like: there are no critical, world-ending bugs left... _that we know of_. Just like any software product, we'll continue to discover critical issues through its entire life cycle. We're still finding occasional critical issues in Drupal 7 almost five years since its first release candidate; that's just a part of supporting a piece of software over the long term. The RC phase means that while Drupal 8 is stable enough to use, we're still discovering critical bugs a little too frequently to recommend it for everyone, in every use case.

"A little too frequently" means that the rate of critical bugs incoming is still too high to be able to promise the fast respond-and-fix turnaround that we want. Every two weeks we'll create a new Release Candidate version that fixes whatever new criticals have been discovered. Once the core team is confident that they can squash bugs in a timely enough manner, they'll (finally) release Drupal version 8.0.0. 

But when will it REALLY be released?
---

"When it's ready" still applies! But we are very, very close now. To give you a point of reference, Drupal 7 went through four Release Candidates before release (two months). That codebase was a lot more fragile than this one, so it's reasonable to hope that we'll have a very Drupally Christmas season this year. Personally I'm betting on January.

<img src="https://www.drupal.org/files/christmas-ad.png">

Can I use it yet?
---

<span style="font-size: 1.5em">*Yes!*</span> <span style='font-size:0.5em'>Some terms and conditions apply.</span>

Just because there are no criticals left, doesn't mean that D8 is completely bug-free! We have [a big pile of known "major" issues](https://www.drupal.org/project/issues/search/drupal?assigned=&submitted=&project_issue_followers=&status[]=1&status[]=13&status[]=8&status[]=14&status[]=4&priorities[]=300&categories[]=1&version[]=8.0.x-dev&issue_tags_op=%3D&issue_tags=) that have been deferred until after 8.0.0, which should impact your decision. You can see at that link that some of them are already ready to be committed. The catch is that during the RC phase, we aren't allowed to commit these fixes. [We're basically only allowed to work on criticals and documentation](https://www.drupal.org/core/d8-allowed-changes#rc). So there are still some serious issues that might be a problem in some use cases.

The biggest issue (that I know of) is [a potential incompatibility between Drupal 8's new "cache tags" header and some hosting providers](https://www.drupal.org/node/2542868). The problem is that Drupal includes some important caching information on the "back of the envelope" of its response to a page request, and it's possible to run out of envelope! If the cache tags header gets too long for the web host to handle, it can behave unpredictably. You might get white screens of death, or it might just shorten the cache tags header, removing important information. There's a solution in the works to allow a maximum length setting, but it won't make it in until 8.0.1 (two weeks after 8.0.0). In the meantime you should avoid D8 if you have any very complex pages with many elements. The examples in that ticket are good ones: a news site with very complex layouts, or a single page site with a lot of "stuff" crammed onto the one, front page.

The other "gotcha" to bear in mind is that it will take some time for Drupal's contributed modules ecosystem to catch up with the new version. According to [Bluespark's status of the top 100 modules for D8](http://www.bluespark.com/status-top-100-contributed-modules-drupal-8) page, so far only 9 of the top 100 D7 modules have a D8 release marked "stable." 19 of those top 100 modules are included in D8 core however, so our total count is up to 28. This is enough to give a good foundation for relatively simple sites, especially if you have some PHP skills under your belt. But I wouldn't go building a complex Intranet on it just yet! 

Wait, so it's still busted?
---

No! Drupal 8 is a solid platform for most use cases - that's the point of the RC release! It's time to go ahead and use it for site builds. Just take it easy and use it for simple sites, first. Give the rest of the community a chance to release stable modules, and hold off on that Facebook-buster behemoth website you've got planned until a few months after launch.

What happens after 8.0.0?
---

After 8.0.0 is released, we will make an enormous, fundamental shift in how Drupal is developed. We will start using [semantic versioning](http://semver.org) with a regular release schedule. Every two weeks we'll release a new "patch level' release: 8.0.1, 8.0.2, and so on. Patch level releases will be bug fixes only, and will be backwards-compatible - that means they won't break anything on your site. Approximately every 6 months, we'll release a new "minor level" release: 8.1.0, 8.2.0, etc. Minor releases are allowed to contain new features, but they are still guaranteed to be backwards-compatible. So even these releases won't break anything on your site. We're still [figuring out]() the exact process for minor releases, but they will include similar phases to what we've seen with D8 core: a beta phase, and release candidates until we're sure there are no more criticals.

What about API changes, and features that would break existing sites? We won't even start developing on those until well into the D8 life cycle. Those changes will belong in the 9.x branch, and will be kept completely separate from anything that could touch your site. 

The key take-away here is that D8 updates should never break your site. They may add features, but they will not interfere with whatever you've already built. We'll continue a regular pace of improving the product in a predictable, scheduled, and backwards-compatible way.

Where are the best Drupal 8 release parties?
---

The Drupal Association is coordinating promotion for official Drupal 8 launch parties. If you want to host one, just [fill out their form](https://assoc.drupal.org/drupal-8-launch-party) and they'll help you promote it! So far no one has built a site mapping the parties, but keep an eye out in the #drupal hashtag on twitter!

Who do I congratulate? Who do I thank?
---

Drupal 8 RC 1 is the combined effort of more than 3200 contributors. That is an incredible number. By comparison, Apache, the world's most popular open source webserver, has 118 contributors. MySQL, the database platform which runs an enormous portion of the Internet, has 1320 contributors. So you can basically walk up to anyone at a Drupalcon and thank him or her!

Most of the contributors to Drupal 8 leaned on the support, training, and hand-holding of mentors at Drupal events all over the world. I know I needed a mentor for my first core contributions, and I got to turn around and mentor other people myself. The mentors are the support network that made this level of mass contribution possible.

But the level of effort is definitely not evenly distributed. Most contributors have made fewer than 20 registered contributions. [But](https://drupal.org/u/dawehner) [some](https://drupal.org/u/tim.plunkett) [people](https://drupal.org/u/berdir) [have](https://drupal.org/u/alexpott) [really](https://drupal.org/u/wim-leers) [gone](https://drupal.org/u/sun) [above](https://drupal.org/u/damiankloip) [and](https://drupal.org/u/xjm) [beyond](https://drupal.org/u/gábor-hojtsy) [what](https://drupal.org/u/larowlan) [anyone](https://drupal.org/u/chx) [would](https://drupal.org/u/andypost) [expect](https://drupal.org/u/ameteescu). [It's](https://drupal.org/u/jhodgdon) [no](https://drupal.org/u/yched) [exaggeration](https://drupal.org/u/joelpittet) [to](https://drupal.org/u/effulgentsia) [say](https://drupal.org/u/yesct) [that](https://drupal.org/u/swentel) [these](https://drupal.org/u/cottser) [people](https://drupal.org/u/nod_) [have](https://drupal.org/u/vijaycs85) [shaped](https://drupal.org/u/pwolanin) [the](https://drupal.org/u/aspilicious) [future](https://drupal.org/u/tstoeckler) [of](https://drupal.org/u/xano) [the](https://drupal.org/u/plach) [Internet](https://drupal.org/u/lewisnyman).

It is easy to concentrate on the number of contributions as the only metric of involvement in the release of D8. But some of the biggest influences on Drupal 8 have been community leaders, whose effort is not counted in commits under their own names. The initiative leads who architected and directed all this contribution: [heyrocker](https://drupal.org/u/heyrocker), [Senpai](https://drupal.org/u/Senpai), [jlambert](https://drupal.org/u/jlambert), [Crell](https://drupal.org/u/Crell), [dmitrig01](https://drupal.org/u/dmitrig01), [Gábor Hojtsy](https://drupal.org/u/gábor-hojtsy), [Jose Reyero](https://drupal.org/u/jose-reyero), [mitchell](https://drupal.org/u/mitchell), [jenlampton](https://drupal.org/u/jenlampton), [bleen18](https://drupal.org/u/bleen18), [jackalope](https://drupal.org/u/jackalope), [ericduran](https://drupal.org/u/ericduran), [jhood](https://drupal.org/u/jhood), [jacine](https://drupal.org/u/jacine), [shyamala](https://drupal.org/u/shyamala), [rupl](https://drupal.org/u/rupl), [JohnAlbin](https://drupal.org/u/johnalbin), [twom](https://drupal.org/u/twom), and [sofiya](https://drupal.org/u/sofiya). Without them, we would have had nothing to commit!

Listing all of those names brings to mind the platform that they all use to contribute and coordinate: [drupal.org](https://drupal.org), maintained by the [Drupal Association](https://assoc.drupal.org/). It also brings to mind the events, like Drupalcon, Drupalcamps, Dev Days, which everyone attends to collaborate, teach, and learn; also maintained by the [Drupal Association](https://assoc.drupal.org/). Not to mention the Drupal 8 Accelerate program, which raised $250,000 towards developer grants; also created and maintained by the [Drupal Association](https://assoc.drupal.org/). The people at the Association have worked tirelessly to support this release.

All of this developer time is extremely valuable, and not all of it came out of the developers' own free time. Huge swaths of Drupal 8 development have been sponsored by the companies that participate in the community. We've only been tracking their contributions for a short time, but the information we have is powerful. This release would not have happened without the developer time donated by companies like [Acquia](https://acquia.com), [MD Systems](http://www.md-systems.ch), [Chapter Three](http://www.chapterthree.com), [Tag1](http://tag1consulting.com), and [Druid](http://druid.fi). A quick glance at [Drupal.org's Drupal Services page](https://www.drupal.org/drupal-services) shows us that contribution is a normal part of the culture for the biggest Drupal companies. These were the top 5, but almost every major Drupal shop has contributed in some measure. Thank you to these companies for believing in our product and supporting it so generously.

Finally, the people who bear the greatest personal responsibility are definitely the core maintainers. These people don't just deserve your thanks; they deserve lifetime supplies of free beer sent to their homes. I can't offer that on a blog; all I can say is THANK YOU.

[Alex Bronstein](https://drupal.org/u/effulgentsia)

[Dries Buytaert](https://drupal.org/u/dries)

[Angie "webchick" Byron](https://drupal.org/u/webchick)

[Nat Catchpole](https://drupal.org/u/catch)

[Jess Myrbo](https://drupal.org/u/xjm)

[Alex Pott](https://drupal.org/u/alexpott)

To everyone who contributed, but especially the people I've listed here: You've made a new generation of Internet innovation possible. Thank you.
