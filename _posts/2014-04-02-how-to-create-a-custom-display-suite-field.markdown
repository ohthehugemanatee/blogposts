---
layout: post
title: "How to create a custom Display Suite field"
date: 2014-04-02 16:12:13 +0100
comments: true
categories: 
 - drupalplanet
 - drupal
 - displaysuite
---
A few months ago I posted about [how to create a custom Panels pane](https://ohthehugemanatee.org/blog/2014/01/03/how-to-create-a-custom-panels-pane), a critical reference for anyone who uses Panels layouts. The other part of the toolkit for quick and awesome layouts is the [Display Suite](https://drupal.org/projects/ds) module. With DS you can create new "Display modes" for your content, to be reused around the site. For example, on one recent site I had four standard ways to display my nodes: Full, Teaser, Mini-Teaser, and Search Result. DS made this configuration a cinch.

But just as in Panels you sometimes need a pane that isn't provided out of the box, in Display Suite you sometimes want to add a field that isn't really a field on your content. In a recent site build, I used this capability to include information from the Organic Groups a user belongs to on his profile as it appears in search results. 

DS offers some ability to create this kind of custom field through the UI, but I'm talking about more complicated outcomes where you need/want to use custom code instead. This is actually even easier than custom panels panes.

In our example, we will display the user's name, but backwards. Obviously you can do much more complex things with this, but it's nice to have a simple example!

Declare your fields
===========

First we have to tell Display Suite about our new custom field. We do this with [hook_ds_fields_info()](http://drupalcontrib.org/api/drupal/contributions!ds!ds.api.php/function/hook_ds_fields_info/7).

```php mymodule.module
<?php

//@file: Add a custom suite to display suite for Users.

/**
 * Implements hook_ds_fields_info().
 * Declare my custom field.
 */
function mymodule_ds_fields_info($entity_type) {
  $fields = array();

  if ($entity_type == 'user') { 
    $fields['backwards_username'] = array(
      'title' => t('Backwards Username'),
      'field_type' => DS_FIELD_TYPE_FUNCTION,
      'function' => 'mymodule_backwards_username',
    );
  return array($entity_type => $fields);
  }
  return;
}
```
Any guesses whathappens next? That's right, we have to write our render function under the name we just declared. You can put anything here, really anything renderable at all. 
``` php mymodule.module
/**
 * Render function for the Backwards Username field.
 */
function mymodule_backwards_username($field) {
  if (isset($field['entity']->name)) { 
    return check_plain(strrev($field['entity']->name));
  }
}
```
That's it. So simple, you'll wonder why you ever did it any other way!
