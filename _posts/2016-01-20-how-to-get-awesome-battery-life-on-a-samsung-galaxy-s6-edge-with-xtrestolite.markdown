---
layout: post
title: "How to get awesome battery life on a Samsung Galaxy S6 Edge with XtreStoLite"
date: 2016-01-20 11:16:38 +0100
comments: true
published: false
categories: 
 - android
---
My Samsung Galaxy S6 Edge died last week, and yesterday I received the replacement. I played a bit with Samsung's own version of Android 5.1.1, but found the same problems that always bug me:

* Bloatware, bloatware everywhere! Samsung has put a lot less crap on the S6 series than in previous incarnations (I'm looking at you, S5!), but there's still an awful lot of "S branded" applications.
* Privacy. The thousand splendid S-applications send a lot of data back to Samsung about me. Not cool. Yes, Android is generally monetized by user data, but that doesn't mean I have to just give mine away to everyone who comes asking!
* Battery life. I got about 4 hours usage time out of a complete charge. That's ridiculous and unacceptable, particularly in a flagship model! The bulk of the battery usage comes from - you guessed it - Samsungs extra apps!

So I set out to install the "de-bloated" version of Samsung's Android, XtreStoLite. Before I go any further I should tell you that I had done this (several times) before, and I know from experience that XtreStoLite can DOUBLE your battery life, before making any extra modifications! So this is the most important step. 

**DISCLAIMER:** I am in no way responsible for any damage you do to your own phone. This post represents my own experience and I hope you can learn from it. It involve modifying your device in a way that voids your warranty. It involves tools that can destroy all your data and keep your phone from booting. So read carefully, and follow along at your own risk!

Odexed or Deodexed? 
---

Before we start, you have to make a choice between two versions of this popular ROM: Odexed or De-odexed. These fancy sounding words describe how Android should store system applications, and the cached information about those apps which it needs to start up. 

Samsung (and most OEMs) distribute their ROMS *odexed*. This means that the system application itself is stored on the ```/system``` partition, which is read-only. More to the point, the cache of information about each system application is kept in separate files, all ready and easy to load on boot. This makes first boot faster, but more importantly for Samsung, it makes it much harder to modify system applications. That's good for security and stability.

Most free public ROMs are distributed *de-odexed*, meaning that the cache about each system application is stored with the application itself. That means that on the first boot-up, Android has to process each system application one at a time, to copy the cache information out of it. After the first boot the cache is already populated and there is no speed difference, but that first startup can take several minutes! The advantage to a de-odexed setup is that it's much easier for developers to modify system apps. 

So if you love Samsung Android just the way it is, you just wish it had a reasonable battery life, then you'll be happy with the *odexed* version. But if there are things about Samsung Android that you hate - like the warning notification when turning up the volume, or the S-Finder and Quick Connect buttons - then you should use the *de-odexed* version.

Since the de-odexed version lets us make tweaks, here's the list of tweaks available:

* Integrated fix for the battery-sucking "no deep sleep" bug
* Removed high volume warning message
* Removed Increasing Ringtone
★ No auto SMS -> MMS conversion with the Samsung messaging app

★ Add secondary symbols to the Samsung Keyboard
★ Native Call recording
★ Camera shutter sound button in the Samsung Camera app
★ Enable All Quick Toggles in the notification panel (will show up after 2nd reboot after an clean install)
★ Allow you to enable the camera during call
★ TouchWiz Launcher is sorted Alphabetically at default
★ Disabled Auto-correction for the Samsung Keyboard at default
★ Filming with the Samsung camera app no longer pauses Music
★ Messaging time stamps to time sent, rather than time received
★ Enable Call Button in call log list
★ Send one message to up to 999 recipients
★ Add an exit button to the Samsung internet Browser
★ Vibrate on call answering
★ Optional: Remove ongoing keyboard switcher notification
★ Optional: Allow Samsung Camera app to run even on low battery
★ Optional: Modded Samsung Messages app to disable screen on when receiving message

You can see that several are optional; you'll get to choose what you like at install time.

For both versions, there is a second step where you can optionally install just the one or two Samsung bloatware apps you actually like. I'm sure there's someone out there. 

Have you picked which one you want? Good! Let's proceed.

Root your phone
---

The first step is to break Samsung's protections around deep system access, or "root" access, to your phone. You bought the phone, so remove the restrictions and control it completely!

You are going to need a Windows computer for this step. I usually try to gear my instructions towards Mac or Linux, but in this case the community really is centered around a Windows-only tool: Samsung's leaked [ODIN](https://en.wikipedia.org/wiki/Odin_%28firmware_flashing_software%29) utility. (NB: there is an open source alternative, [Heimdall](http://glassechidna.com.au/heimdall/). If you get this working with Heimdall, please post instructions in the comments!) It is apparently possible to run ODIN within [Wineskin](http://wineskin.urgesoftware.com/tiki-index.php) on OSX, but that's beyond the scope of this article.

Before we root, we should confirm that the bootloader is unlocked. The bootloader is the very first thing that starts when you press the power button - it's the piece of code that loads everything else. If that sounds important, you're understanding it right. It's also a place where lots of vendors (shittily) will perform double checks to make sure you're running authorized software only. That's called a "locked" bootloader, and if we try to root a device witha locked bootloader, we'll end up with a brick. 

Whether or not your bootloader is locked is going to depend on where you bought your phone. Before you go any further, follow the instructions to [double check if your bootloader is locked or unlocked](https://thebroodle.com/android/how-to-check-your-bootloader-is-locked-or-unlocked/). If you're locked, stop the guide here. Get a new phone, find a way to unlock the one you have, call your carrier and complain, but don't continue with these instructions!

