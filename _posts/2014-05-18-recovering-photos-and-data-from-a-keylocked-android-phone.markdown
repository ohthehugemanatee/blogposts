---
layout: post
title: "How to recover photos and data from a keylocked Android phone"
date: 2014-05-18 18:55:31 +0200
comments: true
categories: 
- android
- hacking
---
Recently a good friend of mine passed away suddenly. Her family asked me to recover her photos and videos from her computer and android phone. The computer was easy enough, but the android phone posed a real problem. It had a keylock enabled, and USB debugging disabled. There was no external SD card to pull out, which meant that all the photos were stored on the internal memory. The phone was a totally stock Samsung Galaxy S3 mini. With a lot of reading, I worked out a way to get a data dump from her phone without having to bypass the keylock.

1) Get Clockworkmod Recovery
---

[Clockworkmod recovery](https://www.clockworkmod.com/) is a familiar tool to anyone who has ever rooted their Android phone or installed a custom ROM. I run my own phone on [Cyanogenmod](http://www.cyanogenmod.org/), so I knew the broad strokes of how to install and use it. If you haven't done this before: your Android device actually has several partitions on it for various kinds of data. Your device actually has several different boot "modes", which are activated by various key combinations on startup. One of the boot modes, "recovery", is specifically there to allow critical system access and maintenance. This is where Clockworkmod lives.

Download the [Clockworkmod ROM for your device](https://www.clockworkmod.com/rommanager). Use the "Touch Recovery" version if your touchscreen is working, since it's the easiest to use.

2) Get a tool to flash your device
---

There are many tools out there to help you flash various partitions on your device. Samsung devices are built for a tool called Odin, which was leaked from the company some time ago. Unfortunately Odin is Windows-only and not all that stable... but there is a great cross-platform, open source alternative called [Heimdall](http://glassechidna.com.au/heimdall/). Get Heimdall, at least, and optionally JOdin3.

Heimdall has an awesome command line interface, and a decent GUI if you're brave and know what you're talking about. I found it much easier to understand by using the Java based (AKA multi-platform) frontend [JOdin3](http://casual-dev.com/2014/01/04/jodin3-web-browser-or-offline-flashing-tool/). Even though in the end I couldn't get JOdin3 to do the job for me, its interface was intuitive enough that I could then understand what was going on from the command line. 

So all you REALLY need here, is Heimdall or the equivalent for your device. But a GUI can be nice.

3) Get Clockworkmod onto your device
---

*NOTE: THE INSTRUCTIONS IN THIS SECTION ONLY APPLY TO SAMSUNG DEVICES. A SIMPLE GOOGLE SEARCH WILL FIND APPROPRIATE INSTRUCTIONS FOR OTHER DEVICES.*

### Get into Download mode

In order for Heimdall/JOdin3/Odin/whatever to connect to your device, it has to be in "Download" mode, sometimes called "Odin" mode. Google will help you find instructions for your specific device, but for the Samsung Galaxy S3 Mini, I had to do the following:

* Power off. You can do this even when the keypad is locked, just by holding down the power button for long enough.
* Press and hold the power, volume down, and home keys simultaneously until the phone starts and a warning screen appears.
* Pay attention to the warning screen, and press volume up when you are ready to continue.

A phone in Download mode looks kinda scary. There's not much on the screen, but it does explain quite clearly that it is in Download mode or Odin mode, and that the current firmware is the stock firmware.

Now plug the device into your computer via USB, and make sure Heimdall can see it by running this on the computer:

{% codeblock %}
sudo heimdall detect
{% endcodeblock %%}

(note: I'm not sure that you really need sudo for these operations, but might as well)

### Figure out which partition you're flashing

In order to update the data on a partition, Heimdall needs to know how the partitions are laid out on your device. Samsung devices can dump their partition table in a format called PIT. Heimdall can actually handle getting the PIT file automatically, but you still need to have a look to see which partition you want to be working on. So first run

{% codeblock %}
sudo heimdall print-pit
{% endcodeblock %}

This will give you a full dump of all the partitions on your device. It's quite readable; look for the partition with the Flash Filename "recovery.img". It should look something like this:

{% codeblock %}
--- Entry #20 ---
Binary Type: 0 (AP)
Device Type: 2 (MMC)
Identifier: 19
Attributes: 5 (Read/Write)
Update Attributes: 1 (FOTA)
Partition Block Size: 491520
Partition Block Count: 32768
File Offset (Obsolete): 0
File Size (Obsolete): 0
Partition Name: Kernel2
Flash Filename: recovery.img
FOTA Filename:
{% endcodeblock %}

*DO NOT SKIP THIS STEP*. In theory all Samsung Galaxy S3 minis are identical, but if yours is not you could brick your device! It's very easy to do.

Figure out the Identifier for that partition. In my case it was "19".

### Flash your recovery partition

Now that you know which partition you want to flash, and you have a file to flash it with, you can actually flash it with heimdall. Note that your phone probably restarted after running that PIT command, so you'll have to get back into Download mode again. Then just run:

{% codeblock %}
sudo heimdall flash --19 recovery.img
{% endcodeblock %}

In that command, "19" is the identifier of the partition I want to flash, and "recovery.img" is the clockworkmod ROM which I downloaded right at the beginning of the post. It will show you step by step what's going on, and then reboot the phone upon success.

4) Connect to the phone with ADB
---

Android has a wonderful tool in the SDK, intended for debugging problems with android devices. It's called ADB, and it lets you connect to the android device over USB and run a variety of commands. It even lets you get into a shell on the device! If you don't have ADB already, download the [Android Developer Tools](https://developer.android.com/sdk/index.html). If you aren't planning on actually developing and just need ADB (like me), then install from the slightly smaller package on that page, listed under "Use an existing IDE".

Once you've downloaded and installed the developer tools, you can find the ADB command in the platform-tools directory from the download.

ADB cannot communicate with a phone unless "enable USB debugging" is turned on in the phone's Settings interface... but when you're locked out that's impossible. Fortunately, if you boot into Clockworkmod Recovery, Debugging is turned on by default. So reboot into Clockworkmod by holding down the power, home, and volume up keys.

When clockworkmod recovery has loaded, test the connection with ADB by running:

{%codeblock %}
adb devices
{% endcodeblock %}

If you see your device listed, you're in business! You can now pull files from the android device with "adb pull [filename] [destination]". Of course, that's assuming you know your way around the android filesystem! 

I found the easiest way to get all the data was to use ClockworkMod's built in backup system to make a backup of all data on the phone (this takes a few minutes), then I could pull the backups because I knew which directory they were in. After running the backup, all I had to do was 

{% codeblock %}
adb pull /data/media/clockworkmod/backup ~/Desktop/android-backup
{% endcodeblock %}

All of my friend's photos, videos, and contacts were stored in the Data tar archive. I hope this helps someone else out there!
