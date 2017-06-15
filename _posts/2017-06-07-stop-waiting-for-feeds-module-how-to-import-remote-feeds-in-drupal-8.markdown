---
layout: post
title: "Stop waiting for Feeds module: how to import RSS in Drupal 8"
date: 2017-06-07 00:33:24 -0400
comments: true
categories: 
 - drupalplanet
 - drupal
 - drupal 8
 - howto
 - migrate
---

How do you import an RSS feed into entities with Drupal 8? In Drupal 6 and 7, you probably used the [Feeds](https://drupal.org/project/feeds) module. Feeds 7 made it easy (-ish) to click together a configuration that matches an RSS (or any XML, or CSV, or OPML, etc) source to a Drupal entity type, maps source data into Drupal fields, and runs an import with the site Cron. Where has that functionality gone in D8? I recently had to build a podcast mirror for a client that needed this functionality, and I was surprised at what I found.

**Feeds module** doesn't have a stable release candidate, and it doesn't look like one is coming any time soon. They're still surveying people about what feeds module should even DO in D8. As the module page explains:

{%blockquote %}
It's not ready yet, but we are brainstorming about what would be the best way forward. Want to help us? Fill in our survey.
If you decide to use it, don't be mad if we break it later.
{% endblockquote %} 

This does not inspire confidence.

The next great candidate is [Aggregator](https://www.drupal.org/docs/8/core/modules/aggregator/overview) module (in core). Unfortunately, Aggregator gives you no control over the kind of entity to create, let alone any kind of field mapping. It imports content into its own Aggregated Content entity, with everything in one field, and linking offsite. I suppose you could extend it to choose you own entity type, map fields etc, but that seems like a lot of work for such a simple feature.

Frustrating, right?

**What if I told you that Drupal 8 can do everything Feeds 7 can?**

What if I told you that it's even better: instead of clicking through endless menus and configuration links, waiting for things to load, missing problems, and banging your head against the mouse, you can set this up with one simple piece of text. You can copy and paste it directly from this blog post into Drupal's admin interface.

What? How?
---

Drupal 8 can do all the Feedsy stuff you like with [Migrate](https://www.drupal.org/docs/8/api/migrate-api/migrate-api-overview) module. Migrate in D8 core already contains all the elements you need to build a regular importer of ANYTHING into D8. Add a couple of contrib modules to provide specific plugins for XML sources and convenience drush functions, and *baby you've got a stew goin'!*

Here's the short version Howto:

**1) Download and enable [migrate_plus](https://drupal.org/project/migrate_plus) and [migrate_tools](https://drupal.org/project/migrate_tools) modules.** You should be doing this with composer, but I won't judge. Just get them into your codebase and enable them. Migrate Plus provides plugins for core Migrate, so you can parse remote XML, JSON, CSV, or even arbitrary spreadsheet data. Migrate Tools gives us drush commands for running migrations.

**2) Write your Migration configuration in text**, and paste it into the configuration import admin page (`admin/config/development/configuration/single/import`), or import it another way. I've included a starter YAML just below, you should be able to copypasta, change a few values, and be done in time for tea.

**3) Add a line to your system cron** to run `drush migrate -y my_rss_importer` at whatever interval you like.

That's it. One YAML file, most of which is copypasta. One cronjob. All done!

Here's my RSS importer config for your copy and pasting pleasure. If you're already comfortable with migration YAMLs and XPaths, just add the names of your RSS fields as selectors in the source section, map them to drupal fields in the process section, and you're all done!

If you aren't familiar with this stuff yet, don't worry! We'll dissect this together, below.

``` yaml
id: my_rss_importer
label: 'Import my RSS feed'
status: true

source:
  plugin: url
  data_fetcher_plugin: http
  urls: 'https://example.com/feed.rss'
  data_parser_plugin: simple_xml

  item_selector: /rss/channel/item
  fields:
    -
      name: guid
      label: GUID
      selector: guid
    -
      name: title
      label: Title
      selector: title
    -
      name: pub_date
      label: 'Publication date'
      selector: pubDate
    -
      name: link
      label: 'Origin link'
      selector: link
    -
      name: summary
      label: Summary
      selector: 'itunes:summary'
    -
      name: image
      label: Image
      selector: 'itunes:image[''href'']'

  ids:
    guid:
      type: string

destination:
  plugin: 'entity:node'

process:
  title: title
  field_remote_url: link
  body: summary
  created:
    plugin: format_date
    from_format: 'D, d M Y H:i:s O'
    to_format: 'U'
    source: pub_date
  status:
    plugin: default_value
    default_value: 1
  type:
    plugin: default_value
    default_value: podcast_episode

```

Some of you can just stop here. If you're familiar with the format and the structures involved, this example is probably all you need to set up your easy RSS importer.

In the interest of good examples for Migrate module though, I'm going to continue. Read on if you want to learn more about how this config works, and how you can use Migrate to do even more amazing things...


Anatomy of a migration YAML
---

Let's dive into that YAML a bit. Migrate is one of the most powerful components of Drupal 8 core, and this configuration is your gateway to it.

That YAML looks like a lot, but it's really just 4 sections. They can appear in any order, but we need all 4: General information, source, destination, and data processing. This isn't rocket science after all! Let's look at these sections one at a time.

**General information**

``` yaml
id: my_rss_importer
label: 'My RSS feed importer'
status: true
```
This is the basic stuff about the migration configuration. At a minimum it needs a unique machine-readable ID, a human-readable label, and `status: true` so it's enabled. There are other keys you can include here for fun extra features, like module dependencies, groupings (so you can run several imports together!), tags, and language. These are the critical ones, though.

**Source**

``` yaml
source:
  plugin: url
  data_fetcher_plugin: file
  urls: 'https://example.com/feed.rss'
  data_parser_plugin: simple_xml

  item_selector: /rss/channel/item
  fields:
    -
      name: guid
      label: GUID
      selector: guid
    -
      name: title
      label: Item Title
      selector: title
    -
      name: pub_date
      label: 'Publication date'
      selector: pubDate
    -
      name: link
      label: 'Origin link'
      selector: link
    -
      name: summary
      label: Summary
      selector: 'itunes:summary'

  ids:
    guid:
      type: string
```
This is the one that intimidates most people: it's where you describe the RSS source. Migrate module is even more flexible than Feeds was, so there's a lot to specify here... but it all makes sense if you take it in small pieces.

First: we want to use a remote file, so we'll use the Url plugin (there are others, but none that we care about right now). All the rest of the settings belong to the Url plugin, even though they aren't indented or anything. 

There are two possibilities for Url's data_fetcher setting: file and http. `file` is for anything you could pass to PHP's [file_get_contents](https://secure.php.net/manual/en/function.file-get-contents.php), including remote URLs. There are some great performance tricks in there, so it's a good option for most use cases. We'll be using `file` for our example. `http` is specifically for remote files accessed over HTTP, and lets you use the full power of the HTTP spec to get your file. Think authentication headers, cache rules, etc.

Next we declare which plugin will read (parse) the data from that remote URL. We can read JSON, SOAP, arbitrary XML... in our use case this is an RSS feed, so we'll use one of the XML plugins. SimpleXML is just what it sounds like: a simple way to get data out of XML. In extreme use cases you might use XML instead, but I haven't encountered that yet (ever, anywhere, in any of my projects). TL;DR: SimpleXML is great. Use it.

Third, we have to tell the source where it can find the actual items to import. XML is freeform, so there's no way for Migrate to know where the future "nodes" are in the document. So you have to give it the XPath to the items. RSS feeds have a standardized path: `/rss/channel/item`.

Next we have to identify the "fields" in the source. You see, migrate module is built around the idea that you'll map source fields to destination fields. That's core to how it thinks about the whole process. Since XML (and by extension RSS) is an unstructured format - it doesn't think of itself as having "fields" at all. So we'll have to give our source plugin XPaths for the data we want out of the feed, assigning each path to a virtual "field". These "fake fields" let Migrate treat this source just like any other.

If you haven't worked with XPaths before, the example YAML in this post gives you most of what you need to know. It's just a simple text system for specifying a tag within an unstructured XML document. Not too complicated when you get into it. You may want to [find a good tutorial](https://duckduckgo.com/?q=xpath+basics) to learn some of the tricks. 

Let's look at one of these "fake fields":

``` yaml
      name: summary
      label: Summary
      selector: 'itunes:summary'
```
*name* is how we'll address this field in the rest of the migration. It's the source "field name". *label* is the human readable name for the field. *selector* is the XPath inside the item. Most items are flat - certainly in RSS - so it's basically just the tag that surrounds the data you want. There, was that so hard?

As a side note, you can see that my RSS feeds tend to be for iTunes. Sometimes the world eats an apple, sometimes an apple eats the world. Buy me a beer at Drupalcon and we can argue about standards.

Fifth and finally, we identify which "field" in the source contains a unique identifier. Migrate module keeps track of the association between the source and destination objects, so it can handle updates, rollbacks, and more. The example YAML relies on the very common (but technically optional) `<guid>` tag as a unique identifier.

**Destination**

``` yaml
destination:
  plugin: 'entity:node'
```
Yep, it's that simple. This is where you declare what Drupal entity type will receive the data. Actually, you could write any sort of destination plugin for this - if you want Drupal to migrate data into some crazy exotic system, you can do it! But in 99.9% of cases you're migrating into Drupal entities, so you'll want `entity:something` here. Don't worry about bundles (content types) here; that's something we take care of in field mapping.

**Process**

``` yaml
process:
  title: title
  field_remote_url: link
  body: summary
  created:
    plugin: format_date
    from_format: 'D, d M Y H:i:s O'
    to_format: 'U'
    source: pub_date
  status:
    plugin: default_value
    default_value: 1
  type:
    plugin: default_value
    default_value: podcast_episode
```

This is where the action happens: the process section describes how destination fields should get their data from the source. It's the "field mapping", and more. Each key is a destination field, each value describes where the data comes from. 

If you don't want to migrate the whole field exactly as it's presented in the source, you can put individual fields through [Migrate plugins](https://www.drupal.org/docs/8/api/migrate-api/migrate-process-plugins). These plugins apply all sorts of changes to the source content, to get it into the shape Drupal needs for a field value. If you want to take a substring from the source, explode it into an array, extract one array value and make sure it's a valid Drupal machine name, you can do that here. I won't do it in my example because that sort of thing isn't common for RSS feeds, but it's definitely possible.

The examples of plugins that you see here are simple ones. `status` and `type` show you how to set a fixed field value. There are other ways, but the `default_value` plugin is the best way to keep your sanity. 

The `created` field is a bit more interesting. The Drupal field is a unix timestamp of the time a node was authored. The source RSS uses a string time format, though. We'll use the `format_date` plugin to convert between the two. Neat, eh?

Don't forget to map values into Drupal's `status` and `type` fields! `type` is especially important: that's what determines the content type, and nodes can't be saved without it! 

That's it?
---

Yes, that's it. You now have a migrator that pulls from any kind of remote source, and creates Drupal entities out of the items it finds. Your system cron entry makes sure this runs on a regular schedule, rather than overloading Drupal's cron.

More importantly, if you're this comfortable with Migrate module, you've just gained a *lot* of new power. This is a framework for getting data from anywhere, to anywhere, with a lot of convenience functionality in between. 

Happy feeding!

Tips and tricks
---

OK I lied, there is way more to say about Migrate. It's a wonderful, extensible framework, and that means there are lots of options for you. Here are some of the obstacles and solutions I've found helpful.

**Importing files**

Did you notice that I didn't map the images into Drupal fields in my example? That's because it's a bit confusing. We actually have an image URL that we need to download, then we have to create a file entity based on the downloaded file, and then we add the File ID to the node's field as a value. That's more complicated than I wanted to get into in the general example. 

To do this, we have to create a pipeline of plugins that will operate in sequence, to create the value we want to stick in our field_image.  It looks something like this:

``` yaml
  field_image:
    -
      plugin: download
      source:
        - image
        - constants/destination_uri
      rename: true
    -
      plugin: entity_generate
```
Looking at that download plugin, *image* seems clear. That's the source URL we got out of the RSS feed. But what is *constants/destination_uri*, I hear you cry? I'm glad you asked. It's a constant, which I added in the source section and didn't tell you about. You can add any arbitrary keys to the source section, and they'll be available like this in processing. It is good practice to lump all your constants together into one key, to keep the namespace clean. This is what it looks like:

``` yaml
source:
  ... usual source stuff here ...
  constants:
    destination_uri: 'public://my_rss_feed/post.jpg'
```

Before you ask, yes this is exactly the same as using the `default_value` plugin. Still, `default_value` is preferred for readability wherever possible. In this case it isn't really possible. 

Also, note that the download plugin lets me set `rename: true`. This means that in case of a name conflict, a _0, _1, _2, _3 etc will be added to the end of the filename.

You can see the whole structure here, of one plugin passing its result to the next. You can chain unlimited plugins together this way...

**Multiple interrelated migrations**

One of the coolest tricks that Migrate can do is to manage interdependencies between migrations. Maybe you don't want those images just as File entities, you actually want them in Paragraphs, which should appear in the imported node. Easy-peasy.

First, you have to create a second migration for the Paragraph. Technically you should have a separate Migration YAML for each destination entity type. (yes, `entity_generate` is a dirty way to get around it, use it sparingly). So we create our second migration just for the paragraph, like this: 

``` yaml
id: my_rss_images_importer
label: 'Import the images from my RSS feed'
status: true

source:
  plugin: url
  data_fetcher_plugin: http
  urls: 'https://example.com/feed.rss'
  data_parser_plugin: simple_xml

  item_selector: /rss/channel/item
  fields:
    -
      name: guid
      label: GUID
      selector: guid
    -
      name: image
      label: Image
      selector: 'itunes:image[''href'']'

  ids:
    guid:
      type: string
  constants:
    destination_uri: 'public://my_rss_feed/post.jpg'

destination:
  plugin: 'entity:paragraph'

process:
  type:
    plugin: default_value
    default_value: podcast_image
  field_image:
    -
      plugin: download
      source:
        - image
        - constants/destination_uri
      rename: true
    -
      plugin: entity_generate

```

If you look at that closely, you'll see it's a simpler version of the node migration we did at first. I did the copy pasting myself! Here are the differences:

* Different ID and label (duh)
* We only care about two "fields" on the source: GUID and the image URL.
* The destination is a paragraph instead of a node.
* We're doing the image trick I just mentioned.

Now, in the node migration, we can add our paragraphs field to the "process" section like this:

``` yaml
  field_paragraphs:
    plugin: migration_lookup
    migration: my_rss_images_importer
    source: guid
```
We're using the `migration_lookup` plugin. This plugin takes the value of the field given in `source`, and looks it up in `my_rss_images_importer` to see if anything with that source ID was migrated. Remember where we configured the source plugin to know that `guid` was the unique identifier for each item in this feed? That comes in handy here.

So we pass the guid to `migration_lookup`, and it returns the id of the paragraph which was created for that guid. It finds out what Drupal entity ID corresponds to that source ID, and returns the Drupal entity ID to use as a field value. You can use this trick to associate content migrated from separate feeds, totally separate data sources, or whatever.

You should also add a dependency on `my_rss_images_importer` at the bottom of your YAML file, like this:

``` yaml
migration_dependencies:
  required:
    - my_rss_images_importer
```

This will ensure that `my_rss_images_importer` will always run before `my_rss_importer`.

(NB: in Drupal < 8.3, this plugin is called `migration`)

**Formatting dates**

Very often you will receive dates in a format other than what Drupal wants to accept as a valid field value. In this case the `format_date` process plugin comes in very handy, like this:

```
  field_published_date:
    plugin: format_date
    from_format: 'D, d M Y H:i:s O'
    to_format: 'Y-m-d\TH:i:s'
    source: pub_date
```

This one is pretty self-explanatory: from format, to format, and source. This is important when migrating from Drupal 6, whose date fields store dates differently from 8. It's also sometimes handy for RSS feeds. :)

**Drush commands**

Very important for testing, and the whole reason we have `migrate_plus` module installed! Here are some handy drush commands for interacting with your migration:

* `drush ms`: Gives you the status of all known migrations. How many items are there to import? How many have been imported? Is the import running?
* `drush migrate-rollback`: Rolls back one or more migrations, deleting all the imported content.
* `drush migrate-messages`: Get logged messages for a particular migration.
* `drush mi`: Runs a migration. use `--all` to run them all. Don't worry, Migrate will sort out any dependencies you've declared and run them in the right order. Also worth noting: `--limit=10` does a limited run of 10 items, and `--feedback=10` gives you an in-progress status line every 10 items (otherwise you get nothing until it's finished!).


Okay, now that's really it. Happy feeding!

{% img center /images/feed-me-seymour.gif "Feed me, Seymour!" %}

