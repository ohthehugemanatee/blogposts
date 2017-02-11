---
layout: post
title: "Perfect Linux Mint on my Macbook Pro 9,2"
date: 2016-10-15 18:44:40 +0200
comments: true
categories:
 - linux
 - osx
---

A few months ago I posted about my [frustrations getting El Capitan working](/blog/2015/10/01/how-i-got-el-capitain-working-with-my-developer-tools/) with my relatively simple developer needs. At the time I complained about the direction Apple is going. One of these days, I threatened, I would jump back to Linux as a primary desktop environment.

This week I read about everyone's problems getting OSX Sierra going, and I decided it was time to take the plunge. Apple, I completely understand the direction you are going with your environment. You've gotta lock down in order to make peoples' computers reliable, easy to use appliances. But I don't want an appliance, I want a computer. I disagree with some of your choices about the system, what software I should use, where I may install it from, and how I may use it. I'm switching to an OS that lets me make those choices.

Someone recently described the choice to me as "program, or be programmed." That really stuck in my mind. OSX increasingly feels like "someone else's computer" for me. Someone else is making all the choices. Well, I don't have to "be programmed" passively. I can run an environment which respects my choices about privacy, which gives me the freedom to do whatever I want with my computer. So this week I went back to Linux.

The last time I used Linux as a primary laptop OS was with Gentoo in 2003. OK, easy to say that this has come a LONG way since then. Still I don't know if I would recommend converting your existing system if you're a techno noob; better to buy a system with Linux pre-configured. With that said, I am damn impressed with how well [Linux Mint](http://linuxmint.com/) runs out of the box on my Macbook Pro 9,2 (mid-2012). Setting everything up was a snap.

How to Get Linux Mint 18 running perfectly on a Macbook Pro 9,2
---

Just install it. Seriously, that's it. It just works.TM

Installation went totally smoothly for me. I am dual booting for now, but I haven't had to get back into OSX all week. Note that you can access all your OSX files from inside Linux, as long as you don't have OSX's "FileVault" disk encryption turned on. Only Macs can decrypt that. There is a similar (more compatible) full disk encryption system for Linux, but at least during the transition I advise you to disable encryption for both "sides" of your computer. I did have to install ReFind to keep the dual boot working.

Linux Mint asks you for permission to use the non-GPL drivers it needs to run your wireless card, and your CPU's special capabilities. It was a checkbox. Everything just worked. I really tried to find anything substantially broken out of the box, and came up empty. Touchpad, sound, webcam, suspend/resume... everything worked just fine. 

Applications
---

I was pleasantly surprised to find that all of my most important applications have Linux flavors available, and they're easy to install. Much easier - or more standardized at least - than installation packages on OSX or Windows! Here are the big ones I use, with any relevant notes.

* SimpleNote
* Firefox Developer Edition
* PHPStorm - sadly I couldn't get it to keep my old licenses. I had to re-enter my license code manually.
* Toggl
* PrivateInternetAccess
* Slack - this app is pretty slow, surprisingly. I may go back to the web version.
* Zoom meetings - still just as painful on Linux
* Skype - skype.com has a newer version of this than the apt repo. No idea what the difference is, but it's still a resource hog.
* Lastpass - browser extension, but deserves a special mention. The version of the extension that's installed automatically (3.3.1) is SLOW and could never connect to Lastpass reliably. Did I mention it's SLOW? It singlehandedly destroyed my system's performance. Visit [the extension's page on addons.mozilla.org]() and scroll down down down until you find "developer channel". There you can download the current version, 4.1.29a. That's fast and light and current and working. 
* FileVault - I keep my most sensitive files in a filevault-encrypted image. Unfortunately, only macs can open that format. I moved everything into a Veracrypt encrypted image instead, as the best multi-platform option.
* Docker - docker "native" for mac is actually running on a very slim Linux VM, so I knew it would be faster on a native Linux machine. But I was surprised that my informal benchmarks clocked it at almost 3x faster! 

I was worried about application coverage, but so far I haven't found any gaps.

Tweaks
---

I had two big tweaks: the task bar, and the touchpad. I also did one minor tweak to improve my battery life.

I actually like the OSX dock and task bar concept. I find the default Windows-style taskbar finicky and a painful way to describe what applications are currently running. So I added [Cairo Dock](https://glx-dock.org/), and configured Cinnamon's built in taskbars to be a bit more OSX'ish.

I'm very finicky about my touchpad's behavior, and Linux Mint's control panel didn't expose enough of the options to me. Fortunately the Mac touchpad runs on the excellent [Synaptics driver](ftp://www.x.org/pub/X11R7.5/doc/man/man4/synaptics.4.html). That means you can configure every detail from the terminal with "synclient optionname=value" or "synclient -l" to list your current settings. Use that man page I linked to tell what options do what, and you can set the perfect configuration. Once you've figured out what options are just right for you, put them in a file called /etc/X11/xorg.conf.d/60-synaptics.conf, like this:

```
Section "InputClass"
  Identifier "touchpad"
  Driver "synaptics"
  MatchIsTouchpad "on"
  MatchDevicePath "/dev/input/event*"
  Option "FingerHigh" "90"
  Option "CoastingSpeed" "7"
  Option "CoastingFriction" "16"
  Option "EmulateTwoFingerMinW" "2"
  Option "RTCornerButton" "0"
  Option "RBCornerButton" "0"
  Option "LTCornerButton" "0"
  Option "LBCornerButton" "0"
  Option "PalmDetect" "on"
EndSection
```

You will probably have to create the xorg.conf.d subdirectory... and you'll have to be sudo for the whole thing.

Finally, I thought I'd try to get more life out of this old battery. So I installed [powertop](https://01.org/powertop), and accepted all of it's automated recommendations. Now I get about 5 hours, which matches my experience in OSX. It is a 4 year old laptop, after all!

That's it. One file to edit from the terminal, press "OK" a bunch on one application, and I got (what feels to me) like a perfect setup. This is so much better than the old days of walking uphill through the snow (both ways!) to get your display working. This is a polished, pleasant to use desktop environment.

I'm happy I switched.


Update October 17 - The hardware CPU fan governor was letting the CPU run very very hot. I finally installed [mbpfan](https://github.com/dgraziotin/mbpfan), and it's under control again. This is all just... so much more painless than I remember!
