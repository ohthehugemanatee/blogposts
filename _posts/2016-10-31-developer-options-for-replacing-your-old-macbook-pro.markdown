---
layout: post
title: "Developer options for replacing your old MacBook Pro"
date: 2016-10-31 09:05:07 +0100
comments: true
categories: 
---
The developer world is abuzz with criticism of the new Macbook Pro. After 4 years of waiting for an update, what we got has [two less ports, slower memory and a worse GPU](https://medium.com/charged-tech/apple-just-told-the-world-it-has-no-idea-who-the-mac-is-for-722a2438389b#.3cl5g99bu), is [a vanity project that the core demographic of Apple’s customers probably won’t even use](https://medium.com/charged-tech/apple-just-told-the-world-it-has-no-idea-who-the-mac-is-for-722a2438389b#.3cl5g99bu), in the final balance: [merely consumer-level](https://petersphilo.org/2016/10/29/open-letter-to-tim-cook/). It's slammed as [incompatible](https://macperformanceguide.com/blog/2016/20161029_1016-Apple2016MacBookPro-compatibility.html), [absurd](https://twitter.com/lapcatsoftware/status/791760906678067200), and [for pros... actually an insult](http://cdm.link/2016/10/apples-computer-vision-looks-backward-others-look-forward/).

If you're a developer in late 2016, and are in the market for a new laptop, there are some great options out there. It's too bad that the latest line of Mac products don't make the cut. I've been neck deep in this research myself, since it's time to retire my mid-2012 non-retina Macbook Pro anyway. I thought I'd share.

I'm looking for a new laptop. It should be portable enough to count as an "Ultrabook", powerful enough to run my IDE and many virtualized environments, with enough battery life to get me through long plane or trane trips. And it should have very high build quality, both for durability and aesthetics.

Operating System
---

OSX made it's name on a great user experience, and that's still mostly true. It's no longer true by such a wide margin, though. Windows has made enormous strides since then - it still feels frustrating for me, but it's worth a try for you. 

Most exciting for an open source guy like me, is the discovery that Linux desktops have gotten great. Gone are the days of futzing around just to get things set up in the first place. With a contemporary Ubuntu or Mint system (at least) you get a pleasant desktop experience out of the box. Give a try to Ubuntu on a live CD, and see how it feels for you. Depending on your mission critical applications, Linux might be a strong choice.

If you find that you *have* to be on OSX, then you're stuck with either Apple hardware, or a [hackintosh](http://www.hackintosh.com/). From personal experience, a hackintosh is a great weekend project, but takes much more work than any Linux system I've used. Every update meant serious surgery. Not for the faint of heart - but if you really need OSX, this might be the best path for you!

Since we've accepted that we're likely moving away from the mac platform, I'm not going to talk much about the operating system here anymore. These systems all (apparently) have good Linux compatibility, so the choice of OS is up to you.


Apple
---

There's a lot (still) to like about Apple products.

* The OS is UNIX-enough-for-most
* The best applications for creatives and developers always have an OSX version
* Build quality is fantastic
* Aesthetics are market leading (ie it's pretty)

Unfortunately, there are down sides too:

* The OS is increasingly "locked in". With every update, you have to boot into recovery mode to disable some "security" features that lock you out of your own machine. 
* For many development applications, you actually need a third party (well maintained) repo of otherwise standard applications - homebrew. 
* The cost! The new MacBook Pro is a very expensive way to buy 4 year old hardware.
* The ecosystem is very clearly moving away from developer needs. From pushing everything to iCloud, to converging Ux with mobile, to disappointing hardware specs, there's no question which way Apple is headed. And it isn't towards better meeting your needs as a dev.


With all this considered, I wouldn't buy a new MacBook for my work tasks. But developers should seriously consider staying with the old Macbook Pro 2012s for another couple of years. Hardware-wise it's still OK, it's just a question of how long before the OS gets too locked down for you.

Looks like you can get one of these babies from eBay for [about $800](http://www.ebay.com/sch/i.html?_from=R40&_trksid=p2050601.m570.l1313.TR12.TRC2.A0.H0.Xmacbook+pro+2012.TRS0&_nkw=macbook+pro+2012&_sacat=0).

Lenovo Thinkpad
---

Thinkpads have been the developer workhorse for years. Durable, with well known and supported hardware, they're especially known for their great Linux support. 

Personally I've always found the build quality lacking in Thinkpads, but I know people who go so far as to custom build and swap out their own components for this versatile beast. Look for models with X, P, or T in the model number, for the best developer oriented models. These babies are extremely customizable - you can get top end hardware, in a well-built and light package. Note the battery life is normally (tested at) 11 hours, and a spare battery pack can bump that up past 24 hours of use. Apple hardware can't even touch that. The spec sheets say that Thinkpads are only fractionally thicker than a Macbook Air, and both models come in at just under 3lbs.

For more information, I can only recommend [Hackaday's recent Thinkpad buyer's guide](https://hackaday.com/2016/10/28/apple-sucks-now-heres-a-thinkpad-buyers-guide/). 


Dell XPS 13 Developer Edition
---

The XPS 13 is another laptop that you see at developer conventions. The developer edition has the option to come with Ubuntu pre-installed, meaning Linux - or at least Debian - compatibility is guaranteed. My colleagues who use XPS 13s love them. 

In most configurations you get a touch screen, and somewhere between 11 and 22 hours of battery life, depending on usage. The only complaint I've heard is the positioning of the webcam. It's on the lower part of the screen bezel, so if you Skype a lot, everyone gets a nice view up your nose. From personal experience, my only complaint is the build quality. It still feels plastic, and just doesn't match up to the machined metal that I've gotten used to.  Still, the XPS 13 Developer Edition is rightfully a leading laptop for serious developers.

Razer Blade Stealth
---

I never thought that Razer would end up on my short list of laptops to consider, but here we are. The [New Razer Blade Stealth](http://www.razerzone.com/gaming-systems/razer-blade-stealth) is a powerhouse machine, in a machined metal chassis that feels every bit as sturdy as a Mac. The processor is the most powerful I've seen in anything ultrabook-sized. Though Razer doesn't officially offer Linux as an option, users on their forums report that Ubuntu works out of the box. 

The only real down side to this machine is the battery, which is rated for just 9 hours of use. I say "just" 9 hours, even though our comparison system got 5-7 hours when it was new. Still, it's something to consider.


Conclusions for now
---

The moment I stepped outside the Apple store, I discovered that there is still a very healthy and diverse market of options for laptop buyers. Every other laptop I considered was cheaper and more powerful than the Macbook Pro line. Every other laptop was more compatible, had more ports, and generally seemed more useful to me. If that's the competitive landscape into which Apple is launching their "absurd" efforts, they must really have abandoned the developer demographic. On the plus side, Linux support is everywhere now. It's possible that this move by Apple will spur a serious wave of Linux adoption among developers.
