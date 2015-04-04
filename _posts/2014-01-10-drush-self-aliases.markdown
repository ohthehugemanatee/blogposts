---
layout: post
title: "Drush self aliases"
date: 2014-01-10 09:22:01 +0100
comments: true
categories: 
  - drupal
  - drupalplanet
  - drush
  - sysadmin
---
I ran into an interesting problem with the drush *@self* alias today. I wanted to pull a fresh copy of the DB down from a client's live site to my local development copy. Should be as easy as *drush sql-sync @clientsite.live @self*, right? I've done this a thousand times before.

And I've also ignored the warning message every time before, but today I thought I'd check it out:

> WARNING:  Using temporary files to store and transfer sql-dump.  It is recommended that you specify --source-dump and --target-dump options on the command line, or set '%dump' or '%dump-dir' in the path-aliases section of your site alias records. This facilitates fast file transfer via rsync.

There are actually two possible solutions to this warning (that I can think of), and they illustrate some of the useful "power user" features of Drush that any frequent user should be aware of.

The warning is there because drush would *prefer* to rsync the DB dump from site1 to site2, rather than a one time copy. Rsync has lots of speed improvements, not the least being diff transfer. When transferring an updated copy of a file which already exists at the destination, rsync will only send over the changes rather than the whole file. This is pretty useful if you're dealing with a large, text based file like an SQL dump - especially one that you'll be transferring often. In order to use this efficient processing though, Drush needs to know a safe path where it can store the DB dump in each location.

First we'll add the *%dump-dir%* attribute to our alias for clientsite:

``` php ~/.drush/clientsite.aliases.drush.php
<?php
// Site clientsite, environment live 
$aliases['live'] = array(
  'parent' => '@parent',
  'site' => 'clientsite',
  'env' => 'live',
  'root' => '/var/www/example.com/public_html',
  'remote-host' => 'example.com',
  'remote-user' => 'cvertesi',
  'path-aliases' => array(
    '%dump-dir' => '/home/cvertesi/.drush/db_dumps',
  ),
);
```

Notice that *%dump-dir* actually goes in a special sub-array for *path-aliases*. This is very likely the only time you'll need to use that section, since most everything else in there is auto-detected. This is the directory on the remote side where drush will store the dump.

Our options come in with the *@self* alias. In a local dev environment, the most common way to handle this is in your *drushrc.php* file:

``` php ~/.drush/drushrc.php
$options['dump-dir'] = '~/.drush/db_dumps';
```

But this won't work for all cases. You can also take advantage of Drush's alias handling by creating a site alias with the settings you want, and letting Drush merge those settings into *@self*. When Drush builds its' cache of path aliases, it uses the site path as the cache key (for local sites only). That means that if you have a local alias with the same path as whatever *@self* happens to resolve to, your alias options will make it into the definition for *@self*. So here's the alternate solution:

``` php ~/.drush/clientsite.aliases.drush.php
$aliases['localdev'] = array(
  'root' => '/Users/cvertesi/Sites/clientsite',
  'uri' => 'default',
  'path-aliases' => array(
    '%dump-dir' => '/home/cvertesi/.drush/db_dumps',
  ),
);
```

There's just one, obscure caveat with the latter method: somewhere in the alias merging process, BASH aliases are lost. That means that '~' stops resolving to your home directory, and you have to write it out (as I did above).

Have fun!
