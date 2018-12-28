---
layout: post
title: "Optimizing data transfer speeds"
date: 2018-12-27 10:30:23 +0100
comments: true
categories: 
  - network
  - sysadmin
---
One of my holiday projects was to set up my home "data warehouse." Ever since [Dropbox killed modern Linux filesystem support](https://www.dropboxforum.com/t5/Syncing-and-uploads/Linux-Dropbox-client-warn-me-that-it-ll-stop-syncing-in-Nov-why/m-p/290065/highlight/true#M42255) I've been using (and loving) [Nextcloud](https://nextcloud.com/) from my home. It backs up to an encrypted [Duplicati](https://www.duplicati.com/) store on [Azure blob store](https://azure.microsoft.com/en-us/services/storage/blobs/), so that's offsite backups taken care of. But it was time to knit all my various drives together into a single RAID data warehouse. The only problem: how to transfer my 2 terabytes (rounded to make the math in the post easier) of data, without nasty downtime during the holidays?

A local network transfer is the fastest, with the least downtime. I have a switched gigabit network in my house, and all my servers are hard wired. That's about 125 megabytes per second; a theoretical 5 hours to transfer everything. Not bad! Start up an rsync and I'm all done! So I kicked it off and went to bed:

``` bash
$ ssh nextcloud.vert
$ rsync -axz /media/usbdrive/ warehouse:/mnt/storage/ --log-file=transfer-to-warehouse.log &
```

I woke up in the morning with the excitement of a kid on Christmas. Everything should be done, right?

``` bash
$ ssh warehouse df -h |grep md0
/dev/md0        2.7T  501G  2.1T  20% /mnt/storage
$
```
Wait, what? How had it only transferred 500 gigabytes overnight? Including time for *Doctor Who* and breakfast, that was only 1 Megabit per second! I knew it was time to play everyone's favorite game: *"where's the bottleneck?* 

I guess it could be rsync scanning all those small files. If that's the case, we'll see high CPU usage, and even higher load numbers (as processes are I/O blocked):

``` bash
$ ssh nextcloud
$ top

top - 08:22:27 up 10:26,  1 user,  load average: 1.20, 1.34, 1.33
Tasks: 170 total,   2 running, 106 sleeping,   0 stopped,   0 zombie
%Cpu(s): 28.0 us,  2.1 sy,  0.0 ni, 69.5 id,  0.1 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem : 16330372 total,   180568 free,   657104 used, 15492700 buff/cache
KiB Swap:  4194300 total,  4162556 free,    31744 used. 15300068 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                  
 8755 ohthehu+  20   0  130572  58456   2672 R  99.0  0.4 513:14.75 rsync                                                                                    
 8756 ohthehu+  20   0   49596   6648   5152 S  16.9  0.0  92:12.29 ssh 
...
```

OK, let's kill the transfer and start again using a single large, piped tarball. No more small file scans!

``` bash
$ ssh nextcloud
$ cd /media/bigdrive && tar cf - . | ssh warehouse "cd /mnt/storage && tar xpvf -"
```

That helps, but we're still compressing lots of data unnecessarily (most of my data is already compressed), and encrypting it, too. We can improve it with a lightweight ssh cipher and disabled compression:

``` bash
$ ssh nextcloud
$ cd /media/bigdrive && tar cf - . | ssh -o Compression=no -c chacha20-poly1305@openssh.com warehouse "cd /mnt/storage && tar xpf -"
```

That chacha20-poly1305 is a very fast cipher indeed - faster than the old arcfour cipher we used to use in this case. But SSH still puts extra work on the CPU. So let's remove it completely from the equation and just use netcat.

``` bash
$ ssh nextcloud cd /media/bigdrive && tar cf - . | pv | nc -l -q 5 -p 9999 
# in a separate terminal
$ ssh warehouse cd /mnt/storage && nc nextcloud 9999 | pv | tar -xf -
```

Transfer speeds now average about 61 megabytes per second. That's fast enough to kick in the law of diminishing returns on my optimization effort: this will take about 8 hours to transfer if I keep it running. I had to pause work for an hour; now if I spend another hour on this, it has to shave more than 25% off my transfer time to finish any earlier tonight. I'm not confident I can beat those numbers.

Still - What happened to my 125 theoretical megabytes per second? Here are the culprits I suspect - and can't really do anything about:

* **Slow disk**: We are writing to a software RAID5 array of old drives. In my head I was using the channel width of SATA-II for my calculations. In reality, and especially on spinning metal, write speeds are much slower. I looked up my component disks on [userbenchmark.com](userbenchmark.com), and the slowest member has an average sustained sequential write speed of 69 MB/s. This is very likely my first bottleneck. At most I can only use half of my available bandwidth.

* **TCP**: After replacing all my drives with SSDs, TCP is the next culprit I would go after. The protocol technically only has about 6% of overhead, but it also dynamically seeks the maximum send rate through [TCP Congestion Control](https://en.wikipedia.org/wiki/TCP_congestion_control). It keeps trying to send "just a little faster", until the number of unacknowledged packets exceeds a threshold. Then it backs off to 50%, and goes back to "just a little faster" mode. This loop means your practical speed with a TCP stream is about 75% of the pipe's theoretical maximum. Think of it like John Cleese offering just one more "wafer thin" packet. I considered using UDP to avoid this, but I actually *want* the error-checking in TCP. Probably the best solution is [something esoteric like UDR](https://github.com/LabAdvComp/UDR).

* **Slow CPU**: This is the last bottleneck here. *Warehouse* is an old Intel Core2 Duo I had lying around the house. Untar and netcat aren't exactly CPU hungry beasts, but at some point there IS a maximum. If you believe the FreeNAS folks, a fileserver needs an i5 and 8 gigs of RAM for basic functionality. I haven't found that to be the case, but then I'm not using RAID-Z, either.

I'm happy with the outcome here. I have another drive to copy later, with another terabyte. I'm considering removing that slowest drive from my RAID array, since the next-slowest one is almost 50% faster. Then I can copy to the array while it's in degraded mode, and re-add the slowpoke afterwards. We'll see.

Happy holidays!


### Appendix: easy performance testing

If you're working on a similar problem for yourself, you might find these performance testing commands helpful. The idea is to tease apart each component of the transfer. There are better, more detailed, dedicated tools for each of these, but in a game of "find the bottleneck" you really only need quick and dirty validation. Fun fact: the command *dd* is actually short for **D**own and **D**irty. Well it should be, at any rate.

**Read speed (on the source** is easy: hand an arbitrary large file to dd, and write down the numbers it gives.

``` bash
$ dd if=large-file.tar.bz2 of=/dev/null bs=1M
1021317200 bytes (1 GB) copied, 3.9888 s, 256 MB/s
```

**Network speed** can be tested by netcatting a gigabyte of zeros from one machine to the other.

``` bash
# On the receiving machine, open a port to /dev/null
$ nc -vvlnp 12345 >/dev/null
# On the sending machine, send a gig of zeroes to that port
$ dd if=/dev/zero bs=1M count=1K | nc -vvn 192.168.1.50 12345
Connection to 192.168.1.50 12345 port [tcp/*] succeeded!
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 11.7811 s, 91.1 MB/s
# Remember, 8 bits to a byte!
$ echo "$(bc -l <<< 91*8) Megabits"
728 Megabits
```

**Write speed on the destination** can be tested with dd, too:

``` bash
$ dd if=/dev/zero bs=1M count=1024 of=/mnt/storage/test.img
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 55.3836 s, 19.4 MB/s
```

(note: these tests were run while the copy was happening on warehouse. Your numbers should be better than this!)
