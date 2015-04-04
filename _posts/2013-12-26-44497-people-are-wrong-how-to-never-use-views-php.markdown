---
layout: post
title: "44,497 people are wrong: how to NEVER need Views PHP."
date: 2013-12-26 12:01:44 +0100
comments: true
categories: 
 - drupal 
 - drupalplanet
 - views
 - howto
---

You're building a View, but you can't get that field to display the way you want it to. Or filter, or sort. Or maybe you have some data in a custom table that you want to include in the View. So you look for a contributed module, and [Views PHP](https://www.drupal.org/project/views_php) looks like the answer to your problem! Until you read the module page, where it says:

>"...it is highly advisable to use regular handlers and plugins when available (or even to create one yourself). Take note that filtering and sorting a view using PHP always has a considerable perfomance impact."

As of this writing, *44,497* site maintainers have read that warning and chosen to ignore it. **They've chosen to put their PHP into a non-revisioned, difficult-to-access place, and to enable PHP input in a module that was never designed for security. They've left their site at risk of a very difficult to diagnose and even harder to fix WSOD**.

I'm going to go out on a limb here, and suggest that in many of these cases, the decision was made because someone had the impression that writing a Views handler or Plugin was difficult. I'm here to tell you that's not so: it's actually quite easy.

What we're doing 
----------------

We're going to tell Views about the structure of the data we want to display, filter, or sort - even if there's not actually a new data source involved, that's how you do it - and then we'll write the function that actually does the filter/sort/etc by improving an existing field display/filter/sort that Views already includes.

This process will work for:

* Defining a new data source for Views, ie something your module keeps in the DB.
* Creating multiple field displays/filters/sorts for an existing field.
* Creating a completely computed field display/filter/sort, with nothing in the DB.

I know that in 99% of use cases for Views PHP, you don't need to define a new data source, table, adn fields. Trust me that this is the easiest way to learn it, though. I promise we'll get to your use case before the end of the post.

How to
======

I'll assume you have a custom module built, with a .info and .module file, but nothing in there yet. We'll call our module "mymodule" for the example.

1) Tell Views about your module
-------------------------------

We implement *[hook_views_api](https://api.drupal.org/api/views/views.api.php/function/hook_views_api/7)* to let Views know that our module provides some code for Views, and what version of the Views API we're using.

``` php mymodule.module
<?php

/**
 * Implements hook_views_api().
 */
function mymodule_views_api() {
  return array(
    'api' => 3,
    'path' => drupal_get_path('module', 'mymodule') . '/views',
  );
}
```

Couldn't be simpler. We declare that we're using Views API 3, and that our Views code will all live in the */views* subdirectory of our module.

2) Tell Views about your custom code
------------------------------------

Now that Views knows to look in our */views* directory, we should populate it. Views will look for a file called *modulename.views.inc* in that directory, so this is where we will put our Views hooks. There are lots of Views interventions you can do in this file, but we're only interested in one: *[hook_views_data](https://api.drupal.org/api/views/views.api.php/function/hook_views_data/7)*.

This hook lets you define new data sources to Views, and for each one show how to render a field, how to Filter results, and how to Sort results based on your new data source. I promised you three use cases up there though, and here's the trick: you don't have to have an actual data source. You can define a filter for a database field that's already described elsewhere.

First let's look at a real field definiton though, because it's simpler. Here's how we would define a real DB table as a data source. The table looks like this:

<table style="border-collapse:collapse;">
<tr><th>naid</th><th>name</th></tr>
<tr><td>1</td><td>Frank Sinatra</td></tr> 
<tr><td>2</td><td>Dean Martin</td></tr>
<tr><td>3</td><td>Sammy Davis, Jr.</td></tr> 
<tr><td>4</td><td>Peter Lawford</td></tr>
<tr><td>5</td><td>Joey Bishop</td></tr>
</table>

So here's our implementation of *hook_views_data*:

``` php views/mymodule.views.inc
/**
 * Implements hook_views_data().
 */
function mymodule_views_data() {
  // Build an array named after each DB table you're describing. In our case,
  // just mymodule_table.
  $data['mymodule_table'] = array(
    // First give some general information about the table as a data source.
    'table' => array(
      // The grouping for this field/filter/sort in the Views UI.
      'group' => t('Example Views Stuff'),
      'base' => array(
        'field' => 'naid', // This is the identifier field for the view.
        'title' => t('Example Views API Data'),
        'help' => t('Names provided by the Mymodule module.'),
      ),
    ),
    // Now we describe each field that Views needs to know about, starting 
    // with the identifier field.
    'naid' => array(
      'title' => t('Name ID'),
      'help' => t("The unique Name ID."),
      'field' => array(
        'handler' => 'views_handler_field_numeric',
        'click sortable' => TRUE,
      ),
      'sort' => array(
        'handler' => 'views_handler_sort',
      ),
      'filter' => array(
        'handler' => 'views_handler_filter_numeric',
      ),
    ),
    // Now the name field.
    'name' => array(
      'title' => t('Name'),
      'help' => t("The Name."),
      'field' => array(
        'handler' => 'views_handler_field',
        'click sortable' => TRUE,
      ),
      'sort' => array(
        'handler' => 'views_handler_sort',
      ),
      'filter' => array(
        'handler' => 'views_handler_filter_string',
      ),
    ),
  );
  return $data;
}
```

This is a pretty simple example, and I think the array structure speaks for itself. First you provide some general information about the table, then you create a sub-array for each field on the table. Each field's array should be named after the field, and provide at least title. Of course it wouldn't be useful if you didn't describe the handlers for any field/sort/filter operations you want to expose. For each one of these you just provide the name of the handler. In this example I used all built-in filters that come with Views, but it's easy enough to provide a custom handler.

Many added behaviors in Views start with *hook_views_data*; this only covers the basics. You can also open fields up as arguments or relationships, or even add built-in relationships. For example, if our table also contained an NID field, we could define a relationship so that node fields are always available when listing names, and vice versa. This stuff is all surprisingly easy to do, it's just not the focus of this post.


3) Write your custom handler
----------------------------

Let's say we want to provide our own field handler for the name field. Maybe we want it to automatically separate first names. This is easy, too! You simply decide on a name for your new handler - by convention it should begin with *modulename_handler_type_*, so we'll use *mymodule_handler_field_firstname*. Here's the relevant part of that *$data* array from before:

``` php /views/mymodule.views.inc
...
    // Now the name field.
    'name' => array(
      'title' => t('Name'),
      'help' => t("The Name."),
      'field' => array(
        'handler' => 'mymodule_handler_field_firstname',
        'click sortable' => TRUE,
      ),
...
```

Not exactly rocket science, is it?

Now we create a file named after the handler, also in the */views* subdirectory. Though you could write your own handler class from scratch, you'll almost never have to. It's much easier to just extend an existing class.

``` php /views/mymodule_handler_field_firstname.inc
<?php

/**
 * @file
 * Definition of mymodule_handler_field_firstname.
 */

/**
 * Provide the first name only from the name field.
 *
 * @ingroup views_filter_handlers
 */
class mymodule_handler_field_firstname extends views_handler_field {
  /**
  * Render the name field.
  */
  public function render($values) {
    $value = $this->get_value($values);
    $return = explode(' ', $value);
    return 'First name: ' . $return['0'];
  }
}
```

You see the pattern we're following: just name a handler, then extend an existing Views handler class to do what you want. You can override options forms, the admin summary... really any aspect of the way Views handles this data. And the pattern is the same for fields, sorts, filters, and arguments.

Once you've created your handler's *.inc* file, you have to make sure your module loads it. So edit your module's *.info* file thusly:

``` ini /mymodule.info
name = My Module
description = Demo module from ohthehugemanatee.org
core = 7.x

files[] = views/mymodule_handler_field_firstname.inc
```

4) Multiple filters for one field
---------------------------------

We all understand how this works for data that you're declaring for the first time in Views. But what if you want to provide multiple handlers for a single field? Maybe there are several different ways to filter or sort it. For most use cases, you should just follow the pattern above, and simply override the Views options form in your handler class. But occasionally you really do need multiple handlers. 

So let's add a second and third field handler for our *name* field:

``` php /views/mymodule.views.inc
...
    // Now the name field. This is the first, and 'real' definition for this field.
    'name' => array(
      'title' => t('Name'),
      'help' => t("The Name."),
      'field' => array(
        'handler' => 'mymodule_handler_field_firstname',
        'click sortable' => TRUE,
      ),
    ),
    'name_last' => array(
      'title' => t('Last name'),
      'help' => t('The Last name, extracted from the Name field'),
      'real field' => 'name',
      'field' => array(
        'handler' => 'mymodule_handler_field_lastname',
        'click sortable' => TRUE,
      ),
    ),
    'name_backwards' => array(
      'title' => t('Evil Genius Name'),
      'help' => t('The name, reversed so it sounds like the name of an evil genius.'),
      'real field' => 'name',
      'field' => array(
        'handler' => 'mymodule_handler_field_evil',
        'click sortable' => TRUE,
        ),
      ),
...
```

Can you spot the difference? All you have to do is add a variable for *real field*, which tells Views the field name to use for the source value, and that's it. Everything else is totally identical to a normal field. By custom we prefix the "virtual" field's name with the name of the real field, but that's as complicated as it gets.

Conclusion
==========

If there's one thing I want you to take away from this blog post, it's that **the Views API is actually really easy**. And if you can't find something online, take a moment to actually look at the [API documentation included with the module](https://api.drupal.org/api/views/views.api.php/). It's *very* thorough, and easy to read. If you feel like you understand how this works, but the doco doesn't quite cover what you're trying to do, look for examples in the Views module itself! There are 169 handlers for every concievable kind of case, just within Views. Find something reasonable and build off of that!

With this in mind, it's only 24 lines of simple code to provide your own handler for an existing field. After that 24 lines, you're doing the same things you were planning to do in views_php... but now you're doing them in a real coding environment, with a revisioning system, and where it's easy to track down and fix errors that could otherwise crash your site. 24 lines of array definition can save you a world of hurt. I hope to see those views_php installation numbers dropping soon.
