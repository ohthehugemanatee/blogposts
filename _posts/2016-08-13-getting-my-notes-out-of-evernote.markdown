---
layout: post
title: "Getting my notes out of Evernote"
date: 2016-08-13 16:44:40 +0200
comments: true
categories:
---

The Why
---

I'm an obsessive note taker. Meetings, ideas, to do lists, grocery lists... The act of writing really helps me remember, so I've been writing everything down since I was a teenager.  For years I had a thousand text documents scattered around my computer. In 2008 I discovered that the act of writing isn't nearly as effective as the act of *searching*, so I resolved to start taking notes in a central, searchable, sync'ed system. In short, I jumped on the Evernote train.

Nowadays, it seems like every time I turn around, Evernote has found some more bloat to add. First it was photos and videos, then to do lists and Reminders. Then came poorly-implemented sharing, groups, chat, geo-tagging, google drive integration, and so on. I recently heard that it does Salesforce integration, too. I'm sure that somewhere out there, there's a target market that wants all of this shit every time they write a shopping list, but I am not in that market. I just want to write, sync, and search text.

I get to digress a little here, because it's my blog. I like simple, reliable tools that don't try to think for me. I write notes in a mix of 4 languages, so spellcheck is an automatic fail. Autocorrect is even worse. Grammar checkers make it impossible to mix short form and long form notes. I don't even want smart quotes. I'm a programmer. When I type something in single quotes, I want it to *stay* in single quotes. In short, if your feature starts with the word "smart", it can fuck off.

In the last several months I've noticed that I have a thousand text notes on my computer again. It's not hard to see why: Evernote has gotten all "smart". OSX's Textedit application can be configured to take the kind of simple notes I like, so that's what I found myself using. So I resolved to find a new note taking system.

The What
---

Here were my criteria:

* ONLY text notes. I've got other ways to handle pictures, drawings, geo-location, and to-do lists. This is just for text.
* The ability to disable everything "smart" in the text editor. I don't even want smart URLs. Just text.
* Automatically saves and syncs. Of the hundred or so text documents on my Desktop, 99% of them are called "Untitled".
* Searchable.
* Accessible from my Phone and all of my computers
* Starting a new note should be as easy as Ctrl+N
* Easy to sync/share with my wife
* Ability to write to a note from the terminal or any other application.

I considered just using vim, but my vim environment is way too feature-packed for what I want. I used vim as an IDE for a long time, remember? I have similar issues with Atom and Emacs - they just do way more than I want them to. Simple, simple, simple!

I chose Simplenote on the recommendation of a few like-minded people, and I love it so far. But of course, there's some migration to take care of!

The How
---

Out of the box, Simplenote is a great note-taking tool for my purposes. I had to disable some of the usual "smart" features, but it took me less than 30 seconds to find and disable them. No biggie. It was missing two key features, though: the ability to write from the terminal, and my long history of Evernote documents.

The solution to both involves an undocumented feature of Simplenote: Dropbox integration. Send an email to customer support and ask for them to enable it on your account. It's rudimentary, and as their support rep explained to me, "this feature isn’t really something we can support, its more of a convenience thing we’ve added for you." I'm cool with that.

Just like that, I have a note-taking system that does two-way sync to a folder of .txt files on all my devices. I can create txt files any way I like, of course: pipe data from the terminal, have PHP write debug output, or use a totally different text editor. That's a huge feature add! It also adds some other nice features, like now my wife can share my notes. Oh, and Spotlight will search those notes automatically, too. Sweet.

Importing from Evernote was a bit of a bitch. Evernote lets you export notebooks to HTML, or HTML wrapped in a custom XML format (.enex, because the world needs another file extension). Neither of those are terribly helpful. I tried a couple of options that failed out on my system:

* [Ever2simple](https://github.com/claytron/ever2simple), which promises to convert .enex into a directory of Markdown text files, wouldn't install because libxml is a tricky beast on OSX. Then it wouldn't run because libxml is a tricky beast on OSX. I gave up.
* [Geeknote](http://www.geeknote.me/), the Evernote command line client, which promises to export to txt files. No installation instructions for non-Linux users, requires python-setuptools which is tricky to get on OSX, and wants sudo access for the installer script without explaining why. Nope.

I finally stumbled on a little PHP project called [enexdump](https://github.com/panicsteve/enex-dump). One file, no installation, written in my most comfortable language after English. Less than 100 lines of code. Perfect! It worked like a charm.

```php enex-dump.php ~/Desktop/my-notes.enex```

And a couple of minutes later, I had an /output folder full of txt files.

So far I can only recommend this setup. Have fun!
