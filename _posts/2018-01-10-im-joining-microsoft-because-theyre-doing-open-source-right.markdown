---
layout: post
title: "I'm joining Microsoft, because they're doing Open Source Right"
date: 2018-01-08 20:32:50 +0100
comments: true
categories: 
  - microsoft
  - job hunt
  - open source
  - drupal
  - drupalplanet
---

I'm excited to announce that I've signed with **Microsoft** as a Principal Software Engineering Manager. **I'm joining Microsoft because they are doing enterprise Open Source the Right Way, and I want to be a part of it**. This is a sentence that I never believed I would write or say, so I want to explain.

First I have to acknowledge the history. I co-founded my first tech company just as the [Halloween documents](https://en.wikipedia.org/wiki/Halloween_documents) were leaked. That's where the world learned that Microsoft considered Open Source (and Linux in particular) a threat, and was intentionally spreading FUD as a strategic counter. It was also the origin of their famous [Embrace, Extend, and Extinguish](https://en.wikipedia.org/wiki/Embrace%2C_extend%2C_and_extinguish) strategy. The Microsoft approach to Open Source only got more aggressive from there, funneling money to [SCO's lawsuits](https://en.wikipedia.org/wiki/SCO/Linux_controversies) against Linux and its users, calling OSS licensing a "cancer", and accusing Linux of violating MS intellectual property.

I don't need to get exhaustive about this to make my point: **for the first decade of my career (or more), Microsoft was rightly perceived as a villain in the OSS world**. They did real damage and disservice to the open source movement, and ultimately to their own customers. Five years ago I wouldn't have even entertained the thought of working for "the evil empire."

Yes, Microsoft has made nice movements towards open source since the new CEO (Satya Nadella) took over in 2014. They open sourced .NET and Visual Studio, they released Typescript, they joined the [Linux Foundation](https://www.linuxfoundation.org/) and went platinum with the [Open Source Initiative](https://opensource.org/), but come on. I'm an open source warrior, an evangelist, and developer. I could see through the bullshit. Even when Microsoft announced the Linux subsystem on Windows, I was certain that this was just another round of Embrace, Extend, Extinguish.

Then I met [Josh Holmes](http://www.joshholmes.com/) at the [Dutch PHP Conference](https://www.phpconference.nl/). 

First of all, I was shocked to meet a Microsoft representative at an open source conference. He didn't even have bodyguards. I remember my first question for him was "What are you _doing_ here?". 

Josh told me a story about visiting startup conferences in Silicon Valley on behalf of Microsoft in 2007, and reporting back to Ballmer's office: 

> "The good news is, no one is making jokes about Microsoft anymore. The bad news is, **they aren't even making jokes about Microsoft anymore**."

For Josh, this was a big "aha" moment. The booming tech startup space was focused on Open Source, so if Microsoft wanted to survive there, they had to come to the table.

That revelation led to the creation of the Microsoft Partner Catalyst Team. Here's Josh's explanation of the team and its job, from an [interview](https://www.youtube.com/watch?v=qkTioWRH-Ws) at the time I met him:

> "We work with a lot of startups, at the very top edge of the enterprise mix. We look at their toughest problems, and we go solve those problems with open source. We've got 70 engineers and architects, and we go work with the startups hand in hand. We'll sit down for a little pair programming with them, sometimes it will be a large enough problem that will take it off on our own and we'll work on it for a while, and we'll come back and give them the code. Everything that we do ends up in Github under typically an MIT or Apache license if it's original work that we're doing on our own, or a lot of times we're actually working within other open source projects."

Meeting with Josh was a turning point for my understanding of Microsoft. This wasn't just something that I could begrudgingly call "OK for open source". This wasn't just lip service. This was a whole department of people that were doing *exactly* what I believe in. Not only did I like the sound of this; I found that **I actually wanted to work with this group**.

Still, when I considered interviewing with Microsoft, **I knew that my first question had to be about "Embrace, Extend, and Extinguish"**. Josh is a nice guy, and very smart, but I wasn't going to let the wool be pulled over *my* eyes.

Over the next months, I would speak with five different people doing exactly this kind of work at Microsoft. I  I did my research, I plumbed all my back-channel resources for dirt. And everything I came back with said **I was wrong**.

Microsoft really *is* undergoing a fundamental shift towards Open Source.

CEO Sadya Nadella is frank that **closed-source licensing as a profit model is a dead-end**. Since 2014, Microsoft has been transitioning their core business from licensed software to platform services. After all, why sell a license once, when you can rent it out monthly? So they move all the licensed products they can online, and rent, instead of selling them. Then they rent out the infrastructure itself, too - hence Azure. Suddenly flexibility is at a premium. As one CTO put it, **for Azure to be Windows-only would be a liability**.

This shift is old news for most of the world. As much as the Hacker News crowd still bitches about it as FUD, this strategic direction has been in and out of the financial pages for years now. Microsoft has pivoted to platform services. Look at their profits by product over the last 8 years:

{% img center /images/microsoft-profits-by-product.png Microsoft profits by product, over year. %}

The trend is obvious: **server and platform services are the place to invest**. Office only remains at the top of the heap because it transitioned to SaaS. Even Windows license profits are declining. This means focusing on interoperability. Make sure *everything* can run on your platform, because anything else is to handicap the source of your biggest short- and medium-term profit. In fact, **remaining adversarial to Open Source would kill the golden goose**. Microsoft *has* to change its values in order to make this shift.

So much for financial and strategic direction; but this is a hundred-thousand-person company. That ship doesn't turn on a dime, no matter what the press releases tell you. So **my second interview question became "How is the transition going?"** This sort of question makes people uncomfortable: the answer is either transparently unrealistic, or critical of your environment and colleagues. Over and over again, I heard the right answer: It's freakin' hard.

MS has more than 40 years of proprietary development experience and institutional momentum. All of their culture and systems - from hiring, to code reviews, to legal authorizations - have been organized around that model. That's very hard to change! I heard horror stories about the beginning of the transition, having to pass every line of contribution past the Legal department. I heard about managers feeling lost, or losing a sense of authority over their own team. I heard about development teams struggling to understand that their place in an OSS project was on par with some Rando Calrissian contributor from Kansas. And I heard about how the company was helping people with the transition, changing systems and structures to make this cultural shift happen.

The stories I heard were important evidence, which contradicted the old narrative I had in my head. **Embrace, extend, extinguish does not involve leadership challenges, or breaking down of hierarchies**. It does not involve personal struggle and departmental reorganization. The stories I heard evidenced an organization trying a real paradigm shift, for tens of thousands of people around the world. It is not perfect, and it is not finished, but I believe that the transition is real. 

**When you accept that Microsoft is trying to reorient its own culture to Open Source, suddenly all those "transparent" PR moves you dismissed get re-framed**. They are accomplishments. It's incredibly difficult to change the culture of one of the biggest companies in the world... but today, almost half of Azure users run Linux. Microsoft's virtualization work made them the [fifth largest contributor to the 3.x Linux kernel](http://www.techradar.com/news/software/operating-systems/inside-the-linux-kernel-3-0-1035353/2). Microsoft maintains [the biggest project on Github (by contributor count)](https://octoverse.github.com/). They maintain a BSD distribution *and* a Linux distribution. And a huge part of LXD (the container-based virtualization system for Linux) comes from Microsoft's work with Canonical.

That's impressive for any company. But Microsoft? It boggles the mind. This level of contribution is not lip-service. You don't maintain a 15 thousand person community just for PR. **Microsoft is contributing as much or more to open source than many other major players, who have had this in their culture from the start** (Google, Facebook, Twitter, LinkedIn...). It's an accomplishment, and it's impressive!

In the group I'm entering, a strong commitment to Open Source is built into the project structure, the team responsibilities, and the budgeting practice. Every project has time specifically set aside for contribution; developers' connections to their communities are respected and encouraged. After a decade of working with companies who try to engage with open source responsibly, I can say that **this is the strongest institutional commitment to "giving back" that I have ever seen**. It's a stronger support for contribution than I've ever been able to offer in any of my roles, from sole proprietor to CTO.

This does mean a lot more work outside of the Drupal world, though. I will still attend Drupalcons. I will still give technical talks, participate, and help make great open source communities for Drupal and other OSS projects. If anything, I will do those things _more_. And I will do them wearing a Microsoft shirt.

Microsoft is making a genuine, and enormous, push to being open source community members and leaders. From everything I've seen, they are doing it extremely well. From the outside at least, **this is what it looks like to do enterprise Open Source The Right Way**.
