---
layout: post
title: "That time I resurrected my Linux MacBook Pro"
date: 2017-06-17 23:06:02 +0200
comments: true
published: true
categories: 
 - linux
 - hacking
---

In the airport last week, my laptop stopped booting.

I have a 2012 Macbook Pro - yes, that coveted "last good model" - and it runs Ubuntu Linux. It's my roving office as I travel the world to conferences, performances, and job sites. So when I started it up at JFK airport this week and got a gray screen of death, I was a little concerned.

I tried starting into Startup Manager by holding the "Option" key. No bootable options appeared. Just an unhopeful file folder with a question mark. This was Not A Good Sign.

{% img center /images/folder-questionmark.jpg When this is all your computer does, it is Not A Good Sign. %} 

Generally this icon means that EFI couldn't find a valid startup device. This is not the first time I've seen this - I've played with my boot options enough to mess this up for myself a few times! I breathed a heavy sigh and tried starting onto the recovery partition by holding "Command+R". I hadn't done anything in particular with Grub recently, but I had done some kernel updates, and you never know... But the recovery boot just got me a circle with a line through it, the universal "no" sign. This was an even worse sign.

{% img center /images/mac-no-sign.png This is an Even Worse Sign. %} 

This immediately threw the issue out of the realm of "something I messed up myself", and into the less-entertaining realm of "hardware failure." 

At home I have some tools for diagnosing this sort of issue, but in the airport I had limited options - and unlimited time. So I started into Apple's built in Diagnostic Test mode by holding "D". A lesser-known feature of the Apple boot manager, is that if it doesn't have a device to boot from, it can download an image from the Internet, load it into RAM, and boot from there. I had never had the chance to try it out before, but that didn't give much satisfaction as I grimly waited for it to work it's magic.

The diagnostic report looked fine, everything in tip-top shape! That is, until I went to the hard drive diagnostic page - there wasn't one. Not just no hard drive diagnostic, but completely no page for SATA diagnostics at all. 

At this point it was clear that a catastrophic hard disk failure was my best-case scenario. Maybe it just skipped the drive diagnostic because no devices reported back, I hoped. There's something zen in that moment, when you find yourself _hoping_ that you've _only_ lost your entire hard drive. All my work is backed up and relatively accessible, after all; I could stare digital death in the eye with only a hundred-Euro bill for a new drive. A motherboard failure, though - I tried not to think of it.

When I got home, I pulled out my toolkit. I opened up my mac to look for signs of burnout or other damage. Nothing. No smell, no scorches, no scratches. Not even any capacitor whine. I have a lot of respect for the engineers who designed a system that, even after 5 years of hard abuse, still retained it's "new motherboard smell." I optimistically tried re-seating the SATA cable for my drive, but wasn't surprised when that didn't work.

I opened my desktop gaming computer, and plugged my laptop hard disk in there. I got the worst result possible from this test: the drive worked fine. It booted into my beautifully customized Linux desktop, dutifully selecting a random background image from the Internet - as luck would have it, a man screaming at his laptop in frustration. Though everyone knows that the silicon gods have a strong sense of irony.

I did a couple of follow up tests to confirm the diagnosis, but it was clear: the hard drive was fine, it was the SATA controller on my Macbook that had died. Everything else seemed intact; I could boot from a USB stick just fine, with a reasonable performance degradation (there's a ~33% speed difference between SATA3 and USB3 interfaces). But it couldn't detect anything connected internally.

The first goal was to get myself a computer for work. I'm a contractor, which means that I don't take days off if I can help it. So I threw together a nice looking machine from spare parts I had laying around the house, and used my ex-laptop hard drive as a primary. When it first booted, I confess to a very satisfying "I told you those parts would come in handy someday" jab to my wife. She was very impressed, I could tell from the way she rolled her eyes at me. :)

My immediate need of a working machine for the office satisfied, the second priority was to buy a new laptop. I'm on the road **a lot**. A desktop working environment - even one that lets me cheerfully tease my partner a little - is a band-aid at best. In fact, my next onsite was in Zürich in two weeks! I briefly considered taking my tower carry-on, before buckling down to find a computer.

My local electronics shop was a bust. They had plenty of beautiful looking machines that would run Linux perfectly nicely - those Lenovos are really coming along! - but they all had one fatal flaw: I can't work on a German keyboard. 

I've lived in Germany for 6 years now; I'm comfortable in the language and culture, and I can complain about the bureaucracy with the best of them. But I just can't handle their terrible keyboard layout. QWERTZ might be fine for writing the language, but code and the *nix terminal are very QWERTY-centric. No, there was no way I would put up with a QWERTZ laptop for my next 5 years.

So I bought one of the popular Dell XPS 13 "developer edition" laptops, with Ubuntu pre-installed. I excitedly checked my order status right away, and saw the shipping date... was the day after I leave for Zürich. Shit.

So my old laptop is back in service, limping along for one last hurrah. The black box on the back looks like some impressive hardware hack, but it's just the old internal drive connected by USB. 

{% img center /images/me-and-my-busted-laptop.jpg It's only a flesh wound! %}

Yes, I had to do the GRUB dance pretty extensively to get it booting. boot-repair is a wonderful tool, but nothing seems to handle elegantly switching between BIOS and EFI boot modes. I ended up having to copy EFI files from my USB stick by hand. As every Linux desktop user ever has said: "if it's working I won't ask any questions."

Yes, I will take this computer into a board room in two weeks.

No, I am not ashamed. I'm actually pretty satisfied with myself. It's been years since I've built a computer myself, let alone rigged one up with duct tape and spare parts. It's been ages since I've had to dig around in the Linux boot process (greatly improved since 2005, Linus be praised). And it's been a long time since I've sported duct tape on my computer.

In a couple weeks I'll find out just how much special sauce Dell has put into their default Ubuntu setup to make it run. My setup is... not very default.
