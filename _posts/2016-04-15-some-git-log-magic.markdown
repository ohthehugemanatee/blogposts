---
layout: post
title: "Some git log magic"
date: 2016-04-15 13:12:41 +0200
comments: true
published: false
categories:
 - bash
 - awk
 - git
---
Today I got to generate git statistics for my team. It's more fun than it sounds! First of all, it's always entertaining to learn just how flexible git's reporting can get. Secondly, it's a chance to dive back into my old sysadmin tool kit and play with awk, sed, and friends.

There were a few questions I wanted to answer about one particular developer's input. I'm concerned that he is working too much after hours. All night crunch sessions are for amateurs, and I'm trying to break him of the habit. Here are my questions:

* What proportion of his commits were generated after hours?
* Which tickets did he work on after hours?
* Which after hours tickets were the biggest, generating the most commits?

First I had to set some standards: "after hours" is defined as "between 7pm and 7am"; ticket ID numbers are noted in our commit log with square brackets, ie ```[#12345]```.  

We'll start with the total number of commits in this last release period, which we need to figure out the proportion of his commits. This is easy with ```git shortlog```. Git log (and shortlog)'s ```--author``` argument matches any part of the user's name, so it was easy enough to get Timmy's commits. This is would be the most intuitive git command I know of, except for the mystery ```-s```: a shortlog-only argument that squashes commits down to a count. 

```
$> git shortlog -s --author=Timmy --after="2016-02-12" --before="2016-04-14"
   609  Timmy Horton
```
(not his real name)

To get the rest of the information we want we have to switch to full git log, and get a custom-formatted one line representation of each commit, with the information that we want. We use ```--format``` to speciy our custom format. Here's [the complete list of format variables](https://www.kernel.org/pub/software/scm/git/docs/git-log.html#_pretty_formats) - our use is pretty trivial.

``` 
$> git log --author=Timmy --format="%cd: %s" --after="2016-02-12" --before="2016-04-14"

Thu Apr 14 00:34:58 2016 -0400: [#50559] Remove dependency on comm_group_stats.inc from foo_communities.info
Wed Apr 13 17:13:02 2016 -0400: [#50549] Add proper EFQs to foo_global_render_homepage_block
Wed Apr 13 17:10:41 2016 -0400: Merge branch 'stable' into 50549-homepage-counts-are-off
Wed Apr 13 00:13:54 2016 -0400: Merge branch 'stable' into 50485-groups-communities-counts
Wed Apr 13 00:13:01 2016 -0400: [#50481] Correct counts calculations on Communities and Groups DS field and Mini Panel Panes
Tue Apr 12 03:59:17 2016 -0400: [#50248] Fix relative url issue on custom DS Author Field
Tue Apr 12 03:18:49 2016 -0400: [#50248] Remove duplicate title in Pathauto alias for Event
Tue Apr 12 03:10:53 2016 -0400: [#50248] Change Pathauto alias for Communities
...
```

Now we'll filter to just the commits which happened between 7pm and 8am, with grep. You can use ```egrep``` to take advantage of "extended" regular expression matching, so I wrote a little regex that looks for the first component of the time, and checks that the second digit is in the right range.

```
$> git log --author=Timmy --format="%cd: %s" --after="2016-02-12" --before="2016-04-14" | egrep '((0[0-9])|(19)|(2[0-3])):\d{2}:\d{2}'

Thu Apr 14 00:34:58 2016 -0400: [#50559] Remove dependency on comm_group_stats.inc from foo_communities.info
Wed Apr 13 00:13:01 2016 -0400: [#50481] Correct counts calculations on Communities and Groups DS field and Mini Panel Panes
Tue Apr 12 03:59:17 2016 -0400: [#50248] Fix relative url issue on custom DS Author Field
Tue Apr 12 03:18:49 2016 -0400: [#50248] Remove duplicate title in Pathauto alias for Event
Tue Apr 12 03:10:53 2016 -0400: [#50248] Change Pathauto alias for Communities
...
```

I'll take this regex apart for you. I know I can recognize a time as any colon-separated trio of two digit numbers. ```\d``` is a number, ```{2}``` is "exactly two of the preceding element", so the pattern is ```\d{2}:\d{2}:\d{2}```. We really want just the hours between 00 and 09, and between 19 and 23. Square brackets denote ranges, but regex examines one _character_ at a time. It sees 01 as a zero, then a one. We have to take it apart a little more than this. The first range is ```0[0-9]``` - that one is easy. We separate it from its neighbor with a vertical pipe to make it an OR comparison, and surround each option in that OR in parenthesis to make sure that both digits are considered together. Since we have to think in single digits, the second time range range could be 19, or ```2[0-3]```. Finally, I surround the whole OR set in parentheses because it makes it easier to read.

Getting a count of this list is easy: we just pipe the result to ```wc -l``` to get a count of lines.

```
$> git log --author=Timmy --format="%cd: %s" --after="2016-02-12" --before="2016-04-14" | egrep '((0[0-9])|(19)|(2[0-3])):\d{2}:\d{2}' | wc -l

225
```

Wow - more than a third of his commits happened after hours! That's definitely not good.

Let's find out which tickets he worked on, using our standardized notation for ticket numbers. We can exclude merge commits with git log's ```--no-merges``` argument. But what about printing just the ticket numbers? We'll use my favorite multitool: awk! 

Awk breaks up input into columns, and has a whole language for doing exotic and fun things with those columns. Who needs excel, anyway? If you want to learn more about awk, check out my [very old post on the subject](https://ohthehugemanatee.org/2011/04/working-with-bash-awk.html). Here we're just going to accept awk's default of space-separated columns, and print off column number 7.

```
$> git log --no-merges --author=Timmy --format="%cd: %s" --after="2016-02-12" --before="2016-04-14" | egrep '((0[0-9])|(19)|(2[0-3])):\d{2}:\d{2}' | awk '{print $7}'

[#50559]
[#50481]
[#50248]
[#50248]
[#50248]
[#50248]
[#50248]
[#50248]
...
```

Great, but there are a lot of duplicates there. I guess 50248 was a big one, for example. So we'll pass it through ```uniq``` to limit it to just unique tickets. Uniq removes duplicates, but only duplicates in a row. So most of the time - including today - we pair it with ```sort```. 

```
$> git log --no-merges --author=Timmy --format="%cd: %s" --after="2016-02-12" --before="2016-04-14" | egrep '((0[0-9])|(19)|(2[0-3])):\d{2}:\d{2}' | awk '{print $7}' | sort | uniq

[#48192]
[#49324]
[#49343]
[#49464]
[#49473]
[#49863]
[#50005]
[#50033]
...
```

Much better! Now, how many commits went into each one of those tickets? This was a new one for me: we'll use uniq's ```-c``` flag to tell it to produce counts. And just for readability, we'll sort the resulting list by the count.

```
$> git log --no-merges --author=Timmy --format="%cd: %s" --after="2016-02-12" --before="2016-04-14" | egrep '((0[0-9])|(19)|(2[0-3])):\d{2}:\d{2}' | awk '{print $7}' | sort | uniq -c | sort

   1 [#50238]
   1 [#50251]
   1 [#50252]
   1 [#50480]
   1 [#50481]
   1 [#50559]
   2 [#50418]
   2 [50033]
   3 [#49863]
   3 [#50319]
...
```

Questions answered! Maybe next I should use gnuplot to plot the number of commits against lateness of the hour...
