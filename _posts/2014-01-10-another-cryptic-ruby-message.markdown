---
layout: post
title: "Another cryptic Ruby message"
date: 2014-01-10 21:20:55 +0100
comments: true
categories: 
 - redmine
 - chiliproject
 - ruby
---
Ruby on Rails drives me nuts, and today I found another example of why: the cryptic error messages. After a server update, my chiliproject/redmine site refused to start. The error message was
> "Object is not missing constant Issue!"

What does that even mean? It took me a little while to understand that *Issue* in this context is the Redmine object "Issue". But still, "Object is not missing constant X" isn't much help. I tried "bundle install" to make sure all my gems were in place, and there was no problem there.

In the end, on a hunch - gah I *hate* fixing things based on hunches - I tried uninstalling the mysql gem. That was the only subsystem which was updated which I could imagine interfering with Redmine. I tried to reinstall it the normal way: *gem install mysql*, which seemed to go without a hitch. But then Redmine insisted that I was missing some gems.

So I tried to use *bundle install* to make sure I didn't lose anything else,  and it just hung at *Fetching source index for http://rubygems.org/*. I don't know why. I have no problem reaching that URL from the server, and it doesn't give me any more information.

Finally I tried skipping the bundle installer, and just specifying the same old version of mysql to reinstall with *gem install --version '= 2.8.1' mysql*. It worked, and I have my PM system back. Huzzah.

One of these days I'm just gonna build a feature-for-feature Redmine clone in Drupal, just out of frustration.
