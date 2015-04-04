---
layout: post
title: "Custom Context conditions"
date: 2013-12-02 09:06:49 -0500
comments: true
categories: 
  - drupal
  - drupalplanet
  - howto 
---
One of the big advantages to using the [Context module](https://drupal.org/project/context "Context Module on drupal.org") is how totally extensible it is. Not only can you use and re-use the built in conditions, you can write your own. This brings all the power of the custom PHP evaluation method of block placement, but in a structure that makes your code re-usable, contributable, versioned, and standards-based. Writing a custom Context Condition is also a great template for how to integrate custom behaviors in many of the more complex Drupal modules such as Views and Search\_API. We'll see this pattern again and again, and this is about the most basic one to demonstrate with.

My task was to determine if the displayed node was entity-referenced as being the "special" node from it's parent organic group. It's a weird requirement (which is exactly why a custom Condition makes sense here), so let me explain that again. On a site with Organic Groups, the Group node has an entityreference field, which marks one of the Group member nodes as special. When the user is viewing this special node, our Rules condition should evaluate to positive.

The first prerequisite is to make absolutely certain that you can't do this using any of the built in Conditions, and something this unique definitely qualifies there. So let's get to the implementation in our custom module. The module will be called CCC for Custom Context Condition.

{% include_code ccc.info lang:ini modules/ccc/ccc.info %}

That's a totally normal .info file, with logical dependencies on OG, EntityReference, and Context modules. Let's have a look at the .module file. This is probably a lot simpler than you expected.

{% codeblock lang:php ccc.module %}
/**
 * Impelements hook_context_plugins().
 */
function ccc_context_plugins() {
  $plugins = array(
    'ccc_condition_og_special_node' => array(
      'handler' => array(
        'path' => drupal_get_path('module', 'ccc') . '/plugins/context',
        'file' => 'ccc_condition_og_special_node.inc',
        'class' => 'ccc_condition_og_special_node',
        'parent' => 'context_condition',
      ),
    ),
  );
  return $plugins;
}
{%endcodeblock %}

First we implement *hook_context_plugins()*, to declare our new condition plugin to Context. This function should return an array of plugins, keyed by plugin name (in our case, ccc_condition_og_special_node). For each plugin, you have to explain to Context some basic information about the handler you're going to write.

* **path** The path to the plugin file. By convention you should put it in your module's directory, under /plugins/context.
* **file** The filename to look for. Keep yourself sane, and name it after the plugin you're writing.
* **class** The name of the Class you're about to write. If you've never written a PHP class before, this is good practice for D8 and object oriented code in general. Think of it like a function name, and again: name it after the plugin you're writing.
* **parent** The Class you are extending to create your condition. If you don't know what to put here, just enter 'context_condition'.

Now that Context knows about your plugin, you have to declare it to the UI in order to use it! For this we implement *hook_context_registry*. This function returns an array keyed by plugin type--in this case, "conditions". For each condition (keyed by condition name), we need title, description, and plugin.

{% codeblock lang:php ccc.module %}
/**
 * Implements hook_context_registry().
 */
function ccc_context_registry() {
  $registry = array(
    'conditions' => array(
      'ccc_condition_og_special_node' => array(
        'title' => t('OG Special Node'),
        'description' => t('Set this context based on whether or not the node is the "Special Node" entityreferenced in the parent OG.'),
        'plugin' => 'ccc_condition_og_special_node',
      ),
    ),
  );
  return $registry;
}
{% endcodeblock %}

Now Context module knows everything it needs to know about your plugin and condition, we have to tell Drupal when to evaluate your condition. You can implement whatever hook make sense for you here, the important part is that you execute your plugin. Since our condition only makes sense after everything else has fired (ie when the OG context is well and firmly set), we'll implement *hook_context_page_reaction()*.

{% codeblock lang:php ccc.module %}
/**
 * Implements hook_context_page_reaction().
 *
 * Executes our OG Special Node Context Condition. 
 * Gotta run on context_page_reaction, so Views and OG have a chance to
 * set/modify Group context. 
 */
function ccc_context_page_reaction() {
  $group = og_context();
  // Only execute the group node context condition if there is a group node
  // in context.
  if ($group) {
    $plugin = context_get_plugin('condition', 'ccc_condition_og_special_node');
    if ($plugin) {
      $plugin->execute($group);
    }
  }
}
{% endcodeblock %}

That's it for your module file. Just declare the plugin to Context and its UI, and find a place to actually execute the plugin. Now we'll write the actual handler class.

Create your plugin file in the place you promised Context to find it in your *hook_context_plugins()* implementation. In our case, this is plugins/context/ccc_condition_og_special_node.inc . We're going to extend Context's basic Condition Class to provide our own functionality. Here are the contents of my ccc_condition_og_special_node.inc file:

{% include_code lang:php modules/ccc/plugins/context/ccc_condition_og_special_node.inc %}

The trickiest part of this is in the Condition settings form and values. Context assumes that your settings form will be a series of checkboxes, and does a lot of internal processing based on that assumption. We don't want to mess any of that up, so there's a bit of dancing around the requirement here.

First we provide the function condition_values. Context needs to know in advance what the possible values are for the Condition's settings form, and this is where you return them. Based on this return, Context will build a settings form of checkboxes for you.

Then we override the settings form with condition_form(). I change the type of the form element to radio boxes, and set a default value.

Then I add my own submit handler, which merely takes the result of the radio box and puts it into an array, just like it would be if this was a checkbox.

Finally, we get to the good part: the execute function. If you recall, this is what we called in *ccc_content_page_reaction()*. Here we load the Group node, and use *entity_metadata_wrapper* to extract the value of the field_special_node entityreference field on that node. Then we test the current NID from the URL. Note that you never have to explicitly return FALSE; Context is only watching for TRUE returns.

When I learned how to do this, I found it surprisingly easy. The hardest part is wrestling with the Condition class to get exactly the behavior you like. Everyone ends up doing *some* dancing around here, so don't feel bad about it. Context's own Conditions are great examples. Have a look at the classes provided in context/plugins/context_condition_*.inc to get ideas for how to do this. 
