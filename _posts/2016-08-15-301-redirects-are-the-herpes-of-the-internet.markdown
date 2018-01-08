---
layout: post
title: "301 Redirects are the Herpes of the Internet"
date: 2016-08-15 11:46:10 +0200
comments: true
categories: 
  - http
  - rant
---
My client bought a domain name based on her favorite username when she was young. Like any teenager, she didn't choose the best name. But SpongeBobLover16 (name changed to preserve her anonymity) wanted to exert her online independence. She decided she would give her real name domain to SpongeBobLover16.com in a 301 redirect. Her parents told her she should always use a 302, but she ignored their advice. "I remember it felt so good," she says today, "but now I have to live with that mistake every day." 5 years later, her real name domain, SusanExample.com, still redirects to that old username on Google. Old friends and family can only see her domain as SpongeBobLover16.com. They say it doesn't bother them, that they love her anyway, but Susan feels marked for life.

Susan's mistake is incredibly common: if you've been a web developer for very long, chances are you have at least one case of a bad 301 in your history. And yet I still see young developers throwing caution to the wind, failing to protect themselves adequately.

The [HTTP RFC](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3.2) is clear that the intention of a 301 is "permanent" moves. A 301 is therefore cacheable forever unless otherwise specified. Most browsers and search engines do just that: they cache the response *permanently*.

In fact, it's a bit deceptive to use "Cache" to describe this behavior at all. "Cache" implies that the record can be cleared, or will otherwise eventually expire. This is not the case for 301 redirects: They are cached *forever* and *irrevocably*. There is no "changing your mind". There is no "circumstances have changed", or "my god that was 15 years ago." When you create a 301, you create it for ever. It's true that most browsers and search engines clear this cache very occasionally - meaning on reinstall, or once every couple of years. But in clearing it even this infrequently, they are technically *violating the spec* by not treating it as a permanent record. This is not behavior you should count on.

So I am constantly surprised by how often I see 301 as a *default* redirect method in software, particularly without a cache expiry header alongside it. This is at its worst with path redirects. A path redirect is temporary almost by definition, at least in the sense of "someday in 5 years I may want to use this path for something else." It's terrible when we talk about domains for a similar reason: very, very few digital decisions are really *permanent*, as in "will survive as long as the Internet" kind of permanent. Users _always_ expect that there's some way to go back, to change the decision someday in the future. But a 301 redirect is a consequence that technically, stays with you for life.

I don't think there's anything wrong with the RFC; as a *capability* the 301 redirect makes perfect sense. But it should be accompanied by dire warnings. I *do* think it's reckless and irresponsible for search engines to encourage this behavior with [documents like this](https://moz.com/learn/seo/redirection). If you're going to encourage 301s, then encourage them with explicit expiry. And if you suspect for even a moment that you're dealing with a web newbie, don't penalize them for being safe and using a 302 "just in case."

Tell your children: always use a 302. Because 301s are forever.
