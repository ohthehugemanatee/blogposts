---
layout: post
title: "Troubleshooting Android error -505 from google play"
date: 2015-05-29 12:20:33 +0200
comments: true
categories: 
 - Android
---
I recently got a new Android phone, and very quickly had trouble installing some of my favorite apps. Google's Play Store downloads the app successfully, then presents me with "Unknown Error -505". 

{% img center /images/error-505.jpg "Error 505" %}

This error message means precisely nothing. It means "something went wrong during installation, and we're not telling you what." This drives me crazy. This blog post is about how to get more information out of that error message, and how to fix the problem.

Your device is not going to give you more information on its own. Instead, you need to install ADB, the "Android Debug Bridge" which lets you type commands directly to your Android device over a USB connection. It's not hard to install, so [go and set that up now](https://duckduckgo.com/?q=how+to+install+adb).

Next, you will have to make sure your Android device permits you to use ADB. On the phone, go to Settings > About Phone, and scroll down to where you see "Build Number" listed. Tap on "Build Number" until you see a popup telling you that you are now a "developer". Then go back to Settings, and tap on the new "Developer Options" menu item near the bottom. Enable the "USB Debugging" option there.

Now plug your phone into your computer via USB, and open up your Terminal. Windows users, use the Console - it's the same thing. First, we'll see if our Android device is even visible to ADB.

``` bash
$> adb devices
```

You should see a list of devices attached, with (probably) only one listing. If not, check to make sure that your phone is really in USB Debugging mode.

Now, we're going to try to install that same troublesome application over ADB. In order to do this, you'll have to download the application's installer (APK) file on your computer. Google makes this hard to do directly, you'll have to use a [third-party website](http://apps.evozi.com/apk-downloader/) or a [browser extension](https://addons.mozilla.org/en-us/firefox/addon/apk-downloader/?src=search) to find your download link. Download that APK file and save it somewhere safe on your computer. Remember the path to that file! I saved mine in *~/Downloads*.

With that APK file in hand, let's try installing your application over ADB. My problematic app was the Deutsche Bahn Navigator app, but obviously you should substitute the filename to your APK here.

```bash
$> adb install ~/Downloads/de.hafas.android.db.apk
5051 KB/s (5991578 bytes in 1.158s)
  pkg: /data/local/tmp/de.hafas.android.db.apk
Failure [INSTALL_FAILED_DUPLICATE_PERMISSION perm=de.dbnavigator.permission.list pkg=de.bahn.dbtickets]
```

(Full disclosure: that is not the exact permission and package I had, I foolishly closed the terminal window before recording it. But the error and the format are correct, which is what's important here.)

NOW we have something to work with! We get an error message, and some information that might explain what's going on. In this case, the installation failed because of a duplicate permission, whatever that means. But it tells me the name of the permission, and - even better - the package that controls it. Notice that com.dbtickets is not the application I was trying to install. It's something else, a separate app I had, called "DB Tickets". This tells me that if I uninstall that other application, it should solve this problem. You can do that from ADB as well:

``` bash
$> adb uninstall de.bahn.dbtickets
Success

```

Now I can try installing the original app again, and it should work. I can do that from the Play Store application, or directly from ADB with the command we tried above. At this point, I'm way more likely to try it through ADB just to make sure I get an error message if it breaks!

You can follow this process to get further information about any error -505 you may receive. You can do all sorts of things through ADB to resolve this kind of problem. Check the [complete list of commands](https://developer.android.com/tools/help/adb.html#commandsummary) for more information.

Postlude
---

This experience pointed out two large Ux failures on the part of Google:

First, is one of the most frustrating Ux concepts out there, which I call: "never present the user with useful information." I understand that 99% of the time, users do not want or need to see technical information about what's going on. Even _I_ want my systems to seem like they "just work." But the 1% of the time when we DO want technical information, is in our error messages. The sensible Ux policy to adopt here, is "make every error message easy to google for more information". 

Second, enabling developer mode. Here are the official instructions: 

> ...go to Settings > About phone and tap Build number seven times. Return to the previous screen to find Developer options at the bottom.

... what? This is 2015. You don't have to hide your "Developer mode" behind some obscure Easter Egg behavior. I don't know if it's standard, but on my phone you're asked right at the start what level of settings you want to see: simple or advanced. Adding a third level for "developer" would make a lot more sense.

But hey, at least we can always get what we want from the terminal.
