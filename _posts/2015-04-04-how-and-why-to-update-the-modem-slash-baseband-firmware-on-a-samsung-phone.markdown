---
layout: post
title: "How and why to update the modem/baseband firmware on a Samsung Android phone"
date: 2015-04-04 08:09:47 -0400
comments: true
categories: 
 - android
 - hacking
---
If you have a Samsung Android phone, and you use Samsung's own version of Android, this article is not for you. When Samsung sends you updates to Android, they also will include any relevant updates to the modem/baseband firmware. So stop reading and go have a cup of tea or something, your job is done.

But for everyone who uses a custom ROM like [Cyanogenmod](http://cyanogenmod.org), [ParanoidAndroid](http://www.paranoidandroid.co/), or [AOKP](http://www.aokp.co/), sit up and pay attention: you're missing some key updates for your phone, and it's probably the reason you have trouble with cellular signal or battery life. I only learned this myself because my beloved Samsung S4 I-9505 has been having a hell of a time keeping signal lately. Over the last 6 months or so, it's picked up a quirk where it randomly drops the signal and starts to overheat in my pocket. Only toggling in and out of airplane mode to restart the connection solves the problem.

Turns out that just as Android gets regular updates, there are updates to the firmware of the cellular modem inside your phone. In any other device, I would expect the firmware to remain extremely stable, and new versions of the OS to include equally stable drivers. In the mobile phone world however, when Google updates Android, manufacturers update their firmware along with it. Mostly these are incremental updates to improve stability or battery life. Over time however, if you're updating your OS but not your firmware, your cellular modem is getting gradually more and more out of sync with what the driver in the OS expects of it.

In my case, I had a recent build of Cyanogenmod based on Android 4.4.4, last updated in March 2015, and a modem firmware that was based on 4.2.2, last updated in May 2013. So my phone was acting up. Maybe yours is doing the same.

Updating your modem firmware doesn't touch the rest of the operating system, and it doesn't touch your data. It's always a good idea to take a backup, but this is not a high risk operation for your data. If you're not careful you might make your signal situation even worse, but at least your photos and contacts will be OK!

Before I go any further, a word about vocabulary: the modem firmware can reasonably be called your "baseband", and sometimes people use "firmware update" to refer to the combined updates to OS _and_ modem firmware that get shipped out by manufacturers. I will only use "firmware" to describe the part that drives the modem, because that's what we're dealing with in this article. On other blog posts or forum posts people will not be so consistent, so read carefully!

In order to update your modem firmware, The first thing to do is to find out the Android version that you're using on your particular ROM. On most ROMs you can find out under "Settings", "About this phone". Look for the line that says "Android Version". 

Next, you have to find out the version of the most recent modem firmware release for your device, built for the same version of Android you're running. [SamMobile](http://www.sammobile.com/devices/) has a fantastic catalog of devices with specifications and build histories for each one. Note that each carrier handles their own updates, so they package their own modem firmwares with slightly different serial numbers. There shouldn't be any significant difference, but check which one comes from your country and provider just in case, using their [Firmware lookup](http://www.sammobile.com/firmwares/). Note that if you want to download from SamMobile, you have to sign up for their download platform. I didn't - instead, I hunted for the right download with Google, and eventually found it on the great [xda developers](http://forum.xda-developers.com) forum. There are a bunch of posts like [this one](http://forum.xda-developers.com/showpost.php?p=50965837) with download links to tons of different firmwares. Make sure you look yours up, first!

If there isn't a build for exactly the same version of Android that you have, it's OK to use one a little bit behind. *Do not* take a modem firmware build that's newer than your Android build. That could leave you in a situation like this;

<iframe width="560" height="315" src="https://www.youtube.com/embed/gVx4OOcIRXg" frameborder="0" allowfullscreen></iframe>

If your phone is a GT-I9505 from T-Mobile Germany, you can shortcut this process and get the same download I did](http://forum.xda-developers.com/showthread.php?t=2192025&page=503). If you have any other device or are from any other region or phone company, *find the right firmware for your own phone*. Double check it, you can really mess up your phone if you do this wrong.

The updated firmware file is a .tar file, which contains two .bin files: modem.bin for the 3g modem in your phone, and NON-HLOS.bin for the LTE modem in your phone. You'll need both. Apparently if you're on Windows you can use [Odin](http://www.downloadodin.info/) which will let you send both those .bin files at once by choosing the combined .tar file. I'm on a Mac so I don't know about that. I used the excellent, open source, cross-platform tool [Heimdall](http://glassechidna.com.au/heimdall/), which has a great, easy command line interface. Download and install, and unpack that .tar file somewhere convenient. 

Then get your phone into "Download mode". For the Samsung Galaxy S4, this means:

1) Power off by holding the power button.
2) Simultaneously hold the home, volume down, and power keys until the phone comes on and asks you if you're SURE you want to go into Download mode. You are sure, so press volume up to continue.
3) Plug the phone into your computer.

Then in the Terminal, run:

``` bash
~$ heimdall detect
Device detected

~$ heimdall flash --APNHLOS ~/Downloads/Modem_GNG8_FULL/NON-HLOS.bin --MDM ~/Downloads/Modem_GNG8_FULL/modem.bin
```
(do not include the ~$ in your typing! It's just an indicator to show you what you type, and what comes back from the system)

"heimdall detect" tells heimdall to find your phone attached via USB. "heimdall flash" tells it to blow away the LTE modem firmware you have now with the file NON-HLOS.bin, and the 3G modem firmware you have now with the file modem.bin.

If all goes well, the phone will reset itself and boot back up normally. You can check to see that your flash worked by looking for the new serial number in your phone Settings. 

Or perhaps you'll find the same problem I did: something else on my computer had already "claimed" this phone, so Heimdall errored out with "libusbx: error [darwin_claim_interface] USBInterfaceOpen: another process has device opened for exclusive access. ERROR: Claiming interface failed!" If you encounter this problem, you can solve it by manually shutting down that other software:

``` bash
~$ sudo kextstat |grep -v apple
Index Refs Address            Size       Wired      Name (Version) <Linked Against>
  134    0 0xffffff7f80eca000 0x4000     0x4000     com.devguru.driver.SamsungComposite (1.4.25) <38 4 3>
  136    0 0xffffff7f80ec3000 0x3000     0x3000     com.devguru.driver.SamsungACMControl (1.4.25) <38 4 3>
  138    0 0xffffff7f80ead000 0xc000     0xc000     com.devguru.driver.SamsungACMData (1.4.25) <84 38 5 4 3>
  141    0 0xffffff7f829fa000 0x6000     0x6000     net.tunnelblick.tun (1.0) <7 5 4 1>
```
Note the lines that say Samsung in them. Those are the Samsung drivers that are blocking Heimdall. Now I hate Kies even _more_! We can shut them down one at a time like this:

``` bash
~$ sudo kextunload -b com.devguru.driver.SamsungComposite
~$ sudo kextunload -b com.devguru.driver.SamsungACMControl
~$ sudo kextunload -b com.devguru.driver.SamsungACMData
```

Then try the heimdall commands again. For my phone, I found greatly improved signal... and because it wasn't spinning extra cycles without signal, greatly improved battery life, too. So we know: every time you update your ROM, do a modem firmware update, too.
