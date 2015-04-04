---
layout: post
title: "How to create a custom Panels Pane"
date: 2014-01-03 13:09:11 +0100
comments: true
categories: 
 - drupal
 - howto
 - drupalplanet
---

Lots of sites are now built with the "Panels everywhere" method, using [Panels](https://www.drupal.org/project/panels) and [Panelizer](https://www.drupal.org/project/panelizer) to configure modular layouts in the Drupal GUI. These modules come with lots of great default Panes, and create even more defaults based on your existing Blocks and Views. But there's always a case for a custom Pane.

As usual, I'll assume that you have an empty custom module called *mymodule*, with only a *.info* and a *.module* file to its name.

1) Tell CTools that you have custom code here
----

Ctools, like Views, needs a hook to declare the fact that you have custom code. To do this we'll use *[hook_ctools_plugin_directory](http://drupalcontrib.org/api/drupal/contributions!ctools!ctools.api.php/function/hook_ctools_plugin_directory/7)*. This hook is invoked for all Ctools plugin types, and includes the module name as a variable. This way you can avoid eating up memory for anything except the targeted module. You also have to declare where your custom code will live. So here's the complete content of *mymodule.module*:

``` php mymodule.module
<?php

/**
 * Implements hook_ctools_plugin_directory().
 */
function mymodule_ctools_plugin_directory($owner, $plugin_type) {
  if ($owner == 'ctools' && $plugin_type == 'content_types') {
    return 'plugins/content_types';
  }
}
```

Note: **Do not confuse Ctools "Content Types" with the "Content Type" entity used elsewhere in Drupal.** This is just confusing naming, but they're totally different things. Actually the most common usage for a Ctools Content Type is a pane, just like what we're doing now. There are other plugin types, but none that interest us in this post.

2) Add your custom pane
---

Oh, did you think this would be more difficult? Now that we've told Ctools to look for Content Type plugins in our module's *plugins/content_types* subdirectory, we just add a *.inc* file for each "Content Type" (aka Pane) that we want to add. Let's do a simple one, which returns the root term of a given taxonomy term. All the following code will go in *plugins/content_types/taxonomy_root_term.inc* (a name I chose arbitrarily).

Right at the top of the file, we provide a *$plugin* array which defines the basic information about our <del>Pane</del> Ctools Content Type. This doesn't go into a function or anything, it just sits at the top of the *.inc* file:

``` php plugins/content_types/taxonomy_root_term.inc
<?php

$plugin = array(
  'single' => TRUE,
  'title' => t('Taxonomy root term'),
  'description' => t('a Display of data from the root term of the given TID'),
  'category' => t('Custom Panes'),
  'edit form' => 'mymodule_taxonomy_root_term',
  'render callback' => 'mymodule_taxonomy_root_term_render',
  'admin info' => 'mymodule_taxonomy_root_term_info',
  'defaults' => array(),
  'all contexts' => TRUE,
);
```

As you can see, this array defines a category, title, and description for the Panels admin interface. It also declares the names of the callbacks which provide the pane's edit form, rendered form, and admin info. "Single" means that this type has no sub-types. This is the case in every single custom pane I've ever seen, so it's probably the case for yours as well.

Now we write the callbacks we named in that array. We'll start with the edit form.

``` php plugins/content_types/taxonomy_root_term.inc

/**
 * Edit form.
 */
function mymodule_taxonomy_root_term($form, &$form_state) {
 $conf = $form_state['conf']; 

 $form['term'] = array(
   '#type' => 'textfield',
   '#title' => t('Term ID'),
   '#description' => t('The term, from which the root term will be displayed'),
   '#default_value' => $conf['term'],
 );

  $entity_info = entity_get_info('taxonomy_term');

  $options = array();
  if (!empty($entity_info['view modes'])) {
    foreach ($entity_info['view modes'] as $mode => $settings) {
      $options[$mode] = $settings['label'];
    }
  }

 $form['view_mode'] = array(
   '#type' => 'select',
   '#options' => $options,
   '#title' => t('View mode'),
   '#default_value' => $conf['view_mode'],
 );

 return $form;
}
```

This is a fairly standard Drupal form. It also goes through typical form validation and submission functions, so you can provide a pretty complete experience for the administrator. In our case, we just want to get the term ID of the term whose root parent should be displayed. We let the administrator enter the term ID, and the view mode which should be used to display it. We won't worry about form validation in our example. Let's move on to the Submit function:

``` php plugins/content_types/taxonomy_root_term.inc
/**
 * Edit form submit function.
 */
function mymodule_taxonomy_root_term_submit($form, &$form_state) {
  $form_state['conf']['term'] = $form_state['values']['term'];
  $form_state['conf']['view_mode'] = $form_state['values']['view_mode'];
}
```

Again, pretty simple stuff. We just make sure that the *$form_state['conf']* has the values entered. Now, the next callback we defined in *$plugin* is for rendering the pane:

``` php plugins/content_types/taxonomy_root_term.inc
/**
 * Render the panel.
 */
function mymodule_taxonomy_root_term_render($subtype, $conf, $args, $contexts) {
  if ($context->empty) {
    return;
  } 
  // Get full term object for the root term.
  $term = ctools_context_keyword_substitute($conf['term'], array(), $contexts);
  $parent_array = taxonomy_get_parents_all($term);
  $root = end($parent_array);

  // Render as a block.
  $block = new stdClass();
  $block->module = 'entity';
  $block->delta = 'taxonomy_term-' . str_replace('-', '_', $conf['view_mode']);

  $entity = entity_load_single('taxonomy_term', $root->tid);
  $block->content = entity_view('taxonomy_term', array($root), $conf['view_mode']);
  return $block;

}
```

First we make sure there is information - ie the taxonomy term ID we need - in the pane's context. Then we get the root term object and render it in the requested display mode. The only requirement for the return here is that it be a [Drupal render array](https://drupal.org/node/930760). So depending on your use case, you can return an image, a field... whatever you like. In most cases a block is a convenient wrapper for whatever you have to return, which is what I did here.

This is as far as you have to go. The admin info callback isn't actually required, just don't include it in the *$plugin* array and you'll be fine. But if you want to make your life easier as a site admin, it's definitely a nice to have.

``` php plugins/content_types/taxonomy_root_term.inc
/**
 * Admin info.
 */
function mymodule_taxonomy_root_term_info($subtype, $conf, $contexts) {
  if (!empty($conf)) {
    $content = '<p><b>Term ID:</b> ' . $conf['term'] . '</p>';
    $content = '<p><b>View mode:</b> ' . $conf['view_mode'] . '</p>';

    $block = new stdClass;
    $block->title = $conf['override_title'] ? $conf['override_title_text'] : '';
    $block->content = $content;
    return $block;
  }
}
```

This just provides the administrative summary which you can see in the Panels UI. Again, Panels will be happy with any render array return you throw at it, so I use a block. 

This is why we have nice things
---

Anyone who's worked with me knows that I'm not a huge fan of the panels everywhere approach. But I use it often, simply because it makes custom layouts and totally custom page pieces so easy to do. Duplicating even this very simple functionality in a block is actually harder than this. You're still using about 3 functions, but you'd have to determine in advance where that TID will come from. It would certainly come out less flexible, and actually probably harder to maintain. With Ctools all your related code sits in one place, and your module structure actually helps you see what's going on where.

If you learn how to do elements like this, you'll find Panels creeping into more and more of your builds. And rightfully so.
