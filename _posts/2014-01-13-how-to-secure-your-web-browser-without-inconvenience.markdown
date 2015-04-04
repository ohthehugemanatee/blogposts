---
layout: post
title: "How to secure your web browser without inconvenience"
date: 2014-01-13 13:36:58 +0100
comments: true
published: false
categories: 
 - Security
 - Convenient Security week
---
The Snowden documents have been a shock to everyone - even the most conspiracy minded were a bit surprised at just how right they _were_. These documents have enormous implications for the politically involved all over the world. From protestor to politician, everyone has that feeling that they're being watched. These people are making radical changes to the way they communicate, and they should.

But what about the rest of us? For people who aren't politically involved, but who just don't like the feeling that their information is being collected everywhere they go? What about the normal people who aren't comfortable having their entire lives documented "just in case"? In other words, what about those of us who want better security, but don't want to give up much convenience for it?

One of the first things you learn in network security is that there is always a tradeoff between security and convenience. Sure you _could_ only use the Internet from public libraries, randomly choosing a new one each time, with a variety of false IDs to get you online. That would be very secure, _but it would also be a huge pain in the ass._ 

Fortunately the inconvenience curve is exponential. That means you can make very significant gains in security before the curve starts getting steep. That means much more privacy, and much more security, without any inconvenience. So long as you aren't the object of any targeted, specific surveillance, you can do a lot to protect yourself.


Assume you're being wiretapped
---

The Internet age has brought an incredible way to make money: aggregating the way people behave online. That means that there's a ton of money waiting for anyone who wants to monitor your Internet connection and report on your behavior. The way the Internet works, it's totally possible and even easy for your Internet Service Provider to wiretap you this way, and sell this information to advertisers.

For example, every time you type a web address into your computer, it has to perform a "lookup" to find out what server that address refers to. That lookup is performed by checking with your ISP. This is a lot like the old telephone '411' system. You want to contact Bob's movie theater, so you call your telephone company and ask for their phone number. In the Internet age, your telephone company can make a huge amount of money on the side by keeping a record of every 411 request you made, and selling that list to advertisers. This is a very simple kind of data collection that is practiced industry wide, called DNS logging.

Another common practice is actually logging the details of your web traffic. Anything with a URL that begins with 'http://' is sent and received in plain text, so your ISP can read every page you visit. They can analyze all that information and sell the results.

These are not complicated for an ISP to implement, they offer a great side income, and they're totally within your ISP's rights. They are practiced by most major commercial Internet Service Providers. They are definitely practiced by any public wifi service like Starbucks.

Oh, and it's worth noting: any time you connect to a Wifi system that doesn't require a password to connect, all your communications are actually being _broadcast_ to anyone within Wifi range. So any airport wifi, or the Internet at your friend's house who never bothered to set up a password... you are literally broadcasting the contents of every page you visit on a radio. (if they require you to login in a web browser, it makes no difference).

Maybe you're outraged by this, maybe not. Either way, the smart response is to simply assume you're being wiretapped. That's the bad news. The good news is, it's easy to protect yourself so that even someone with a wiretap can't get any useful information. To continue the telephone analogy, this is like speaking in code. You know that anytime you're speaking in code, a broad based wiretap program wouldn't be able to discern your meaning.


Step 1: Install an open source web browser
---

One of the biggest things we learned from the Snowden leaks is that large organizations can and do put monitoring and cracking backdoors into proprietary (closed source) products. They get some into open source products as well, but at least in open source these backdoors get discovered and fixed as bugs. In propriety software, tough luck.

This rules out Internet Explorer, Safari, Chrome, and Opera. So *my recommendations go to Firefox and Chromium*. [Firefox](http://getfirefox.com) is a very popular browser: fast, secure, and totally customizable. [Chromium](http://www.chromium.org/) is the open source project that provides much of the code in Google's closed-source Chrome browser. Google runs the project, and doesn't offer downloads of the actual browser... just the source code. But [you can download Chromium here](http://chromium.woolyss.com). Think of it as an open source version of Chrome.

Either one is a big improvement. I personally prefer Firefox because of the amazing extensions and support, but I won't hate on you if you go Chromium. Unfortunately all the plugins I'm recommending in this guide will be Firefox centric though, so you'll have to leave a comment with the best Chromium-compatible alternatives.

Changing web browsers to an open source alternative offers a great improvement in privacy and security without inconvenience.

Step 2: Set up a VPN
---

A VPN is a system that "tunnels" your Internet connection to another spot somewhere in the world, and makes it seem like all your web traffic is coming from there. Everything in the "tunnel" is encrypted, so even someone who is wiretapping you can't see it. This is the only way to use a public Wifi hotspot securely, and should be a part of your regular practice.

In the old days, a VPN used to slow down your connection greatly. Modern commercial VPN providers offer very fast connections, and in 99% of cases you won't notice the difference. Almost all of them come with applications that will automatically sign you on to the VPN every time you connect to the Internet.

My personal recommendation goes to [Private Internet Access](https://www.privateinternetaccess.com/pages/buy-vpn/). They're the cheapest that I know of, they provide fast enough connections that you can use P2P systems like Bittorrent without noticable slowdown, and they let you choose what country your "out point" should be in. For $7/month ($3.30/month if you pay for a year at once), it's hard to go wrong. 

No matter which VPN provider you use, you should be using one at all times.
 Setting up an always-on VPN offers a great improvement in privacy and security without inconvenience.

Step 3: Use a real password manager
--- 

Most users on the Internet rotate fewer than 4 passwords for all their various services and sites. This is an excellent way to have your identity or your credit card stolen. You should understand that the way hacking works in most cases is guessing passwords. Humans are slow at guessing and typing passwords, but a machine can try thousands of different passwords per second.  Most browsers and operating systems offer a built-in password manager. You should know that *most of these are not very secure.* A real password manager plugs into every web browser you use, on every device, and provides a _locally encrypted_ store of passwords for all your sites. Most of them offer auto-complete into websites, as well as strong password generation.

Broadly, the way this works is that you set a very, very strong password for your password management system. This is the one password that will control them all, so it should be a phrase with numbers, symbols, and caps. A password management system doesn't just remember and auto-fill all your passwords for you: it generates them as well. See, you don't have to remember any passwords anymore, so you can let the machine make them as ridiculously long and complex as you like.

Thanks to my password management system, all of my passwords are 25 character, random combinations of mixed caps letters, numbers, and symbols. I don't know what any of them are, but my manager's auto-complete is actually easier to use than the browser's built in one.

A password manager is a massive improvement to your security, and it actually makes the Internet *more* convenient to use.

Step 4: Install privacy browser plugins
---

This is a list of the (all free) privacy plugins that I use, which don't inconvenience you at all but *do* improve your privacy and security:

* [Adblock Plus](https://adblockplus.org/): This plugin blocks content - banners, popups, tracking scripts, video ads, etc - from any known advertiser, on all the websites you visit. It's totally silent and runs in the background, so there's no convenience cost.

* [CipherFox](https://addons.mozilla.org/en-US/firefox/addon/cipherfox/): When you see "https://" in the URL you're safe, right? Wrong. There are many ways to implement HTTPS, not all of them secure. This plugin lets you click on the lock icon in your browser bar, and get a summary of the HTTPS information... including a quick link to an HTTPS security test from Qualys labs. This is a must-have in my opinion.

* [Disconnect](https://disconnect.me/): Facebook doesn't just track what you do on facebook.com. They also track everything you do on any website with a "like" button. Disconnect blocks this, and more than 2,000 different tracking systems from all your websites. Webpages load up to 27% faster, and you can rest easy that advertisers aren't tracking your every move anymore.

* [Google Sharing](http://www.googlesharing.net/'): This plugin automatically intercepts anything you send to google's public services (Maps, Search, etc... anything that doesn't contain your private information like Mail), and makes sure Google can't build a coherent profile on you. mixes everyone's requests into a big pool,  
