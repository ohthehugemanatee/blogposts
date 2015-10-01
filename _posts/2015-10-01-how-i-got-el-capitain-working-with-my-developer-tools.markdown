---
layout: post
title: "El Capitain broke my developer stuff! Here's how to fix it"
date: 2015-10-01 12:44:42 +0200
comments: true
published: true
categories: 
- developers
- system
- OSX
- troubleshooting
---
Last night I installed OSX's new "El Capitain" update, and it broke most of the tools I use in my daily life as a developer, including homebrew and git. Here are the steps I had to follow to get everything working again.

1) Disable System Integrity Protection
---

[System Integrity Protection](https://en.wikipedia.org/wiki/System_Integrity_Protection) is a new feature of OSX, also called "rootless." As the nickname suggests, it creates another level of access above your root account. The nerfed root account can't modify anything in a (broad) list of system files, folders, or processes. This makes sense for most users, who blindly enter their sudo password for any installer that asks. It is not so good for developers, who regularly use these folders. Homebrew is one application that expects access to */usr/local*, a protected folder. Other applications that are busted by SIP are OSXFuse, MacGPG, Git... the list goes on. So step one is to disable SIP.

* Reboot into recovery mode (hold Command+R on reboot)
* Access the Terminal from the dropdown menus.
* Run `csrutil disable`
* Reboot back into OSX

Congratulations, your root account is un-nerfed. If you use OSXFuse, a VPN, or anything else that relies on unsigned KEXTs, you'll have to keep this disabled. 


2) Fix permissions on /usr/local
---

Open the terminal and run:

``` bash
sudo chflags norestricted /usr/local && sudo chown $(whoami):admin /usr/local && sudo chown -R $(whoami):admin /usr/local
```

This fixes the permissions to what homebrew and other applications expect: you own the directory and all its contents. It also changes the flags for SIP, to allow future modifications if you choose to re-enable it. 

3) Fix developer tools
---

I also found that every time I tried to run.... well almost anything, I would get errors like this:
``` 
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
Stashing your changes:
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
Error: Failure while executing: git stash save --include-untracked --quiet
```

To fix this, you have to re-install Xcode developer tools, like so:
``` bash
xcode-select --install
```
It took 5 minutes to download and install on my system.

4) Fix homebrew
---

To make sure your homebrew install is working again:

``` bash
#$ brew update
#$ brew doctor
```

Follow any instructions the good doctor gives you! Here's what I did:

``` bash
brew link ppl011
brew install libxml2 onigurama
```

Those were probably more to do with how badly I maintain my own homebrew cellar than anything else, but if the doctor gives you instructions, follow them!

Finally, update your brew-installed applications - many of them have updates for El Capitain.

``` bash
brew upgrade
```


5) Fix git
---

You're not using the outdated version of git that comes with Xcode developer tools(2.3.8), are you? No, you're using the current release (2.5.3), [downloaded from git-scm.com](http://git-scm.com/download/mac). Whether you had it previously installed, or just installed it now, it lives in /usr/local/git . But the system will still default to using the old Xcode version. Here's how to fix that:

``` bash
#$ git --version
git version 2.3.8 (Apple Git-58)
#$ sudo mv /usr/bin/git /usr/bin/git.xcode
Password:
#$ sudo ln -s /usr/local/git/bin/git /usr/bin/git
```

6) Re-install GPGTools
---

The GPGTools team have done a great job of handling the known incompatibility between GPGMAil and El Capitain's version of the Mail application... but I still had trouble with the other parts of the suite (which I actually use). I found it easiest to just [download and re-install](https://gpgtools.org).

7) Optional, but recommended: tell me about good Linux laptops
---

I understand how SIP is helpful for normal consumers, but developers are not normal consumers. This step by Apple crosses my comfort barrier, so I'm officially interested in a linux laptop for the first time in years. I have a while to decide, but if you know of a Linux laptop that can compete with my Macbook Air in terms of convenience, sexiness, and reliability, please post about it in the comments. 

That's everything I had to do to get my system back in working order... for now. I'll update this post if I find more steps! 
