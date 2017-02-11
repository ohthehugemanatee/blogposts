---
layout: post
title: "How to Migrate content into Drupal 8 Paragraphs"
date: 2016-08-27 10:52:16 +0200
comments: true
published: false

categories:
  - drupal
  - howto
  - drupalplanet
  - drupal 8
---
[Paragraphs module](https://drupal.org/project/paragraphs) is rapidly becoming a critical tool in every Drupalist's site building toolkit. It creates a field that references a (draggable) list of paragraph entities. Paragraphs are an entity type with bundles, which means you have all sorts of different paragraph types to choose from, each with its own customizable fields, layout, template, etc.

Though it originally started as a way to address "long form" content pieces, today Paragraphs is used as a fundamental building block for much more. I talked to a colleague last week who uses it for component-based design: *everything* on his sites is a paragraph. Every page layout is just a content type, with a Paragraphs field for each column. All components are treated equally. So the library of components that you built in a good, component-based design process, is translated into a library of Paragraphs, representing everything from a front page slideshow, to callout boxes, to sidebar blocks. Pretty cool stuff.

This growth in use cases for Paragraphs means that Drupal 8 migrations almost inevitably involve Paragraphs. They can be a bit tricky, too. It's an entity reference field, but a little too complicated to use the entity_generate plugin. Plus, it's really an [Entity Reference Revisions](https://www.drupal.org/project/entity_reference_revisions) field, so you need some revision handling, too.

I've seen some people handle this through a complicated [migrate process plugin](https://www.drupal.org/node/2129651). That's a little counter-intuitive to me, and you end up with a fair bit of custom code. The simplest solution I've seen is to give Paragraphs their own Migration.
