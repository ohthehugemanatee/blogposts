---
layout: post
title: "My War on Systemd-resolved"
date: 2018-01-25 11:05:31 +0100
comments: true
categories: 
 - linux
 - ubuntu

---

<p>I run ubuntu as the base for my daily driver machine &ndash; heavily customized though it is &ndash; because Canonical&rsquo;s choices are, by definition, mainstream. That makes them easy to support, easy to understand, and generally easy to work with. So what I&rsquo;m about to describe is exceptional in how frustrating it is for me. Seriously, this one issue keeps is enough to drive me into the arms of another distro.</p>

<p>Ubuntu has a built in DNS cache, which it checks first when trying to resolve anything. This Makes Sense For The User in that DNS queries are resolved faster if they come from local cache. It Makes Sense for the network admin, in that repetitive DNS queries don&rsquo;t take up bandwidth. But it really <strong>doesn&rsquo;t</strong> Make Sense for the web developer.</p>

<p>The local DNS cache takes up port 53, which is a problem if you&rsquo;re trying to run any different kind of DNS service locally. For example, the DNS service that practically any integrated docker-based development environment (<a href="https://docksal.io/">docksal</a>, <a href="https://github.com/drud/ddev#ddev">ddev</a>, <a href="https://docs.amazee.io/local_docker_development/pygmy.html">pygmy</a>, etc etc etc) needs. Even my own haproxy-based setup needs to control DNS. Not to mention, <a href="https://superuser.com/questions/1153203/ubuntu-17-04-systemd-resolved-dns-lookups-randomly-fail">the system randomly fails</a> all on its own! So: in order to do my job, I need to disable the Ubuntu DNS proxy. That&rsquo;s where the trouble begins.</p>

<p>There are a hundred variations on <a href="https://askubuntu.com/questions/898605/how-to-disable-systemd-resolved-and-resolve-dns-with-dnsmasq">how to disable systemd-resolvd</a> on askubuntu and stackoverflow. All of them have different suggestions. Partly this is just the flexibility of *nix systems. But it&rsquo;s largely because in the 5 years since Ubuntu introduced the local DNS cache, it&rsquo;s changed which system it&rsquo;s using (twice), how it starts up, how its config is stored, and the overall startup daemon control process. Every year, there are new tricks to learn for how to disable this totally extraneous system. And every time you update, it is magically re-enabled (or enabled in a new way). This is enough that it&rsquo;s kept me from bothering with dist-upgrades for the last year. So I run the latest kernel version, but an old version of the OS, partly because I really hate fighting this problem.</p>

<p>For my own notes, and hopefully to help some others, here is how I disable the local DNS cache on Ubuntu 17.04.</p>

<figure class='code'><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
<span class='line-number'>7</span>
<span class='line-number'>8</span>
<span class='line-number'>9</span>
<span class='line-number'>10</span>
<span class='line-number'>11</span>
<span class='line-number'>12</span>
<span class='line-number'>13</span>
<span class='line-number'>14</span>
</pre></td><td class='code'><pre><code class=''><span class='line'># confirm that it's running
</span><span class='line'>$ sudo netstat -tulpn |grep 53
</span><span class='line'>udp        0      0 127.0.0.53:53           0.0.0.0:*                           28168/systemd-resolved
</span><span class='line'>$ sudo ps -aux |grep resolved
</span><span class='line'>systemd+ 28168  0.0  0.0  50024  5300 ?        Ss   Jan24   0:01 /lib/systemd/systemd-resolved
</span><span class='line'># try to kill it.
</span><span class='line'>$ sudo killall systemd-resolved
</span><span class='line'>$ ps -aux |grep resolved
</span><span class='line'>systemd+ 28168  0.0  0.0  50024  5300 ?        Ss   Jan24   0:01 /lib/systemd/systemd-resolved
</span><span class='line'># roll your eyes, and try to disable it through the normal startup routes.
</span><span class='line'>$ sudo service resolvconf disable-updates
</span><span class='line'>$ sudo update-rc.d resolvconf disable
</span><span class='line'>$ sudo service resolvconf stop
</span><span class='line'># pray that it survives a reboot.</span></code></pre></td></tr></table></div></figure>


<p></p>

<p>Note that if you do this, you have to manage your own nameservers in /etc/resolved.conf! In my case networkmanager does it for me, but that won&rsquo;t be true for everyone.</p>
