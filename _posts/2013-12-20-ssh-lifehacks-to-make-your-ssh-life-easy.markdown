---
layout: post
title: "SSH lifehacks to make your SSH life easy"
date: 2013-12-20 16:29:27 +0100
comments: true
categories: 
 - sysadmin
---
If you use SSH every day, or even every other day, this post is for you. We're going to walk through some of the more convenient options available to you in your global SSH configuration to make your life easier and faster.

First of all, you should know that SSH has a great configuration file, typically located in your home directory at *.ssh/config*. Everything we're talking about today belongs there. There's tons of stuff you can do with it, and I'm only going to scratch the surface.


Re-Using Existing Connections
----

Every time you SSH into a server, it opens a new connection and authenticates all over again, right? That's slow, and it's a PITA if you have to type a password for your private key or just to access the server. You're already connected once, so why not reuse the same connection? Just add these lines to your *.ssh/config*:

```
# Re-use existing connections instead of opening new ones
ControlMaster auto
ControlPath /tmp/ssh_mux_%h_%p_%r_%l
```

This tells SSH to use a socket file for each connection, and to name the file in a way that's unique for each combination of host, port, remote username, and local hostname. That is to say, if you're connecting to the same host/port with the same username and from the same local hostname, it will automatically re-use the existing connection. 

**Result: Multiple SSH connections are WAY faster.**

Host shortcuts and definitions
---

You can easily define commonly used connections with shortcuts. So instead of typing `ssh -i ~/.ssh/ohthehugemanatee.pem -P 2222 ohthehugemanatee@live.ohthehugemanatee.org`, you can just type `ssh live`.

``` bash
Host live 
  Hostname live.ohthehugemanatee.org 
  Port 2222
  User ohthehugemanatee
  IdentityFile ~/.ssh/ohthehugemanatee.pem

```

SSH Agent Forwarding - One key to rule them all
---

Normally when you want to access a resource from multiple machines, you have to generate a public/private keypair on each machine. A great example is a git repo: if I want to access the same git repository on my localhost, the staging server, and the live server, I will have to generate three keypairs and grant access to all three. Talk about a pain in the ass! 

Fortunately there's a much easier way to do this, which is available in almost all distributed versions of openssh: SSH agent forwarding. With agent forwarding enabled, your private key from one machine becomes available for each subsequent machine you connect to. In other words, if you have one private key on your localhost, and SSH into another server, your local private key will be available if you want to SSH (or use git) from that server. Automatically. And only while you're connected, so it's safe.

```
ForwardAgent yes
```

Just drop that in your Host definition (per above), and if the remote host supports it it will work automatically. It is not recommended to set this option globally, because your private key is actually very sensitive stuff, and you don't want to accidentally forward your key to an untrusted server. But feel free to enable it on every host where it's convenient!

Bonus material
-------------

Technically these items aren't about *.ssh/config*, but they're still so convenient I had to share them. 

Note that the same improvements to your SSH will also apply to your use of SCP and other SSH tools. So that brutally long and painful rsync-over-ssh command that you've had to type a thousand times, can now just use "live" as a shortcut. Git, too. It's a great relief to be able to type `scp ~/Sites/index.php live:~`. It's just so much more readable!

But if you want to get **even lazier**, you're going to love the next tip. Why ssh in at all, when you can just mount the remote directory somewhere convenient on your local system? Let's use that Host alias above, and mount the remote web root (*/var/www/html*) at *~/live-environment*. It's much more convenient there, after all.

`sshfs live:/var/www/html ~/live`

Note that this no longer works in OSX without installing [OSX Fuse](http://osxfuse.github.io/). Still, the 5 minute download and install process is totally worth it for this easy SSH lifehack.

Or is that not lazy enough for you? "Do I really have to mount the whole directory?" I hear you cry! "I wanna just edit one file..." Well have I got the tip for you! For the ultimately lazy, [vim](http://www.vim.org/) supports editing files directly over SSH. Try it out:

`vim scp://live/var/www/html/index.php`

Think about it: before you read this post, that would have taken you several long-winded commands to type. Your arthritic fingers can thank me later.
