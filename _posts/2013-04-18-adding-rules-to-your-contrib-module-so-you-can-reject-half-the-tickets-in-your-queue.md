---
layout: post
comments: true
title: Adding Rules to your contrib module (so you can reject half the tickets in
  your queue)
created: 1366306354
categories: 
 - drupal
 - drupalplanet
---
<p><a href="http://drupal.org" target="_blank">Drupal's</a> <a href="http://drupal.org/project/rules" target="_blank">Rules module</a> lets site builders define complex and custom behaviors without having to code a custom module. That puts a lot of power in the hands of the site builder, and it makes developers (rightly) nervous. But there's another way to think about it.</p><p>What's fantastic about Rules is that it lets everyone who wants your module to behave slightly differently to <em>change the behavior for themselves</em>. In fact, if you build the logic of your module into Rules you get&nbsp;an excuse to reject half the tickets in your contrib module queue on the grounds of "works as intended," aka "do it yourself." &nbsp;To me, this is the quiet brilliance of Commerce module's dependence on Rules. Their code is almost 100% functional elements, with as little behavioral logic as possible. All of the logic is written in Rules. Of course they still distribute Commerce with a hefty set of default rules, so it works with a standard implementation out of the box. But at the same time, it is open to completely custom workflows without the developers ever having to get involved.</p><p>Today we're going to cover adding Rules to existing code. This is an easy way to write solid and useful patches for other contrib modules out there, patches that get used. For your own projects you will probably want to implement this code in your own custom module, just hooking into core or contrib behaviors. Either way the logic is the same.</p><p>At it's core, the Rules API lets you expose functions you've already written to the GUI. 90% of the work is just declaring the functions. There are three elements that Rules cares about:</p><ul><li><strong>Events</strong>: Potential triggers that a user can use to take action with Rules.</li><li><strong>Actions</strong>: Functions which take a set of parameters and do something.&nbsp;</li><li><strong>Conditions</strong>: Test functions, which return <span style="font-family:courier new,courier,monospace;">TRUE</span> or <span style="font-family:courier new,courier,monospace;">FALSE</span>.</li></ul><p>You declare these elements to Rules with hooks, usually assembled into a separate <span style="font-family:courier new,courier,monospace;">modulename.rules.inc</span> file. First, let's define an event with <span style="font-family:courier new,courier,monospace;"><a href="http://drupalcontrib.org/api/drupal/contributions!rules!rules.api.php/function/hook_rules_event_info/7" target="_blank">hook_rules_event_info()</a></span>.</p>
<pre class="brush: php; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="mymodule.rules.inc">&lt;?php
/**
* Implements hook_rules_event_info().
* Fire a Rules event when a node is validated.
*/
function mymodule_rules_event_info() {
&nbsp; return array(
&nbsp;&nbsp;&nbsp; 'mymodule_node_is_being_validated' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'group' =&gt; t('My module'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'label' =&gt; t('A node is being validated!'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'variables' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'node' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'type' =&gt; 'node',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'label' =&gt; t('unsaved node being validated'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ),
        'form_id' =&gt; array(
          'type' =&gt; 'text',
          'label' =&gt; t('Form ID'),
        ),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ),
&nbsp;&nbsp;&nbsp; ),
  );
}
?&gt;
</pre>
<p>This implementation tells rules that you are defining an Event with the machine name <span style="font-family: 'courier new', courier, monospace;">mymodule_node_is_being_validated</span>. In the Rules dropdown select box that lets you choose the triggering action for a new rule, your Event is called "A node is being validated!" and is sorted into the group "My Module". &nbsp;The variables array sets the variables that are offered with this particular event. Each variable in the array must have a &nbsp;machine name (the key), a human-readable label and a data type. Now, anywhere that you want to fire this rules event, you call it with&nbsp;<span style="font-family: 'courier new', courier, monospace;">rules_invoke_event('mymodule_node_is_being_validated', $node)&nbsp;</span>to fire the Event. In this case, you could hook into node validation and add it easily enough:</p>
<pre class="brush: php; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="mymodule.module">&lt;?php
/**
* Implements hook_node_validate().
* Fires our Rules event on node validation.
*/
function mymodule_node_validate($node, $form, &amp;$form_state) {
  rules_invoke_event('mymodule_node_is_being_validated', $node, $form['#form_id']);
}
?&gt;
</pre>
<p>This example brings up one of the big limitations of Rules: you can't fail validation. We can set an error message, respond with all the power of Rules, but no matter what you do you cannot make this node form fail validation. The good people at <a href="http://drupal.org/project/rules_forms" target="_blank">rules_forms</a> module are trying to address this, but it's a much thornier problem than it looks.</p><p>Next, let's define a Rules action with <span style="font-family:courier new,courier,monospace;"><a href="http://drupalcontrib.org/api/drupal/contributions%21rules%21rules.api.php/function/hook_rules_action_info/7" target="_blank">hook_rules_action_info()</a></span>. This time we'll use a real world use case from a recent client: our Rules Action will allow you to subscribe a user to a node with the <a href="http://drupal.org/project/subscriptions" target="_blank">subscriptions</a> module. Again, we'll add this info implementation to our <span style="font-family:courier new,courier,monospace;">mymodule.rules.inc</span> file for clarity.</p><p>&nbsp;</p>
<pre class="brush: php; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="mymodule.rules.inc">&lt;?php
/**
* Implements hook_rules_action_info()
* Add a Rules Action for subscribing a user to a node.
*/
function mymodule_rules_action_info() {
&nbsp; return array(
&nbsp;&nbsp;&nbsp; 'mymodule_subscribe_user_to_a_node' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'label' =&gt; t('Subscribe a user to a node'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'group' =&gt; t('Subscriptions'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'parameter' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'user' =&gt; array(
          'type' =&gt; 'user',
          'label' =&gt; 'Subscribing user',
        ),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'node' =&gt; array(
          'type' =&gt; 'node',
          'label' =&gt; 'Target node',
        ),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ),
&nbsp;&nbsp;&nbsp; ),
&nbsp; );
}
?&gt;
</pre>
<p>&nbsp;</p><p>You can see that we use the same <span style="font-family:courier new,courier,monospace;">label</span>, <span style="font-family:courier new,courier,monospace;">group</span>, and <span style="font-family:courier new,courier,monospace;">module</span> declarations here. Actions take parameters as well, which are structured very similarly to the <span style="font-family:courier new,courier,monospace;">variables</span> in events. You don't have to do anything more to make an action: this is it. When Rules fires the action it will look for a function called <span style="font-family:courier new,courier,monospace;">mymodule_subscribe_user_to_a_node</span>, and fire it with the parameters provided in the GUI. In this case we wrapped our own function around subscribe module's functionality, because of the way subscribe's own subscription function works.</p><p>&nbsp;</p>
<pre class="brush: php; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="mymodule.module">&lt;?php
/**
* Creates a subscription for the user
*
* @param $user
*&nbsp; The user to subscribe to the node.
* @param $node
*&nbsp; The node to which the user will subscribe.
*/
function mymodule_subscribe_user_to_a_node($user, $node) {
&nbsp; module_load_include('inc', 'subscriptions', 'subscriptions.admin');
&nbsp; if (!is_object($node) || !isset($node-&gt;nid)) {
&nbsp;&nbsp;&nbsp; watchdog('rules_subscriptions', 'Error: object passed is not node. Data: !node', array('!node' =&gt; print_r($node, true)), WATCHDOG_ERROR);
&nbsp;&nbsp;&nbsp; return;
&nbsp; }
&nbsp; $form_state = array(
&nbsp;&nbsp;&nbsp; 'values' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'stype' =&gt; 'node',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'sid' =&gt; $node-&gt;nid,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'uid' =&gt; $user-&gt;uid,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'author_uid' =&gt; NULL,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'send_interval' =&gt; _subscriptions_get_setting('send_interval', $user),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'updates' =&gt; _subscriptions_get_setting('send_updates', $user),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'comments' =&gt; _subscriptions_get_setting('send_comments', $user),
&nbsp;&nbsp;&nbsp; ),
&nbsp; );
&nbsp; drupal_form_submit('subscriptions_add_form', $form_state, 'node', $node-&gt;nid);
}
?&gt;
</pre>
<p>&nbsp;</p><p>So much for Rules actions - they're even easier than events! &nbsp;There is one big defining difference for Actions though: they can provide new variables back to Rules. This is as easy as a <span style="font-family:courier new,courier,monospace;">provides</span> array in <span style="font-family:courier new,courier,monospace;">hook_rules_action_info()</span>, and returning an array of values from the actual function called.&nbsp;</p><p>Now let's look at a Rules condition. No surprise by now, we declare the existence of our Rules Condition through <span style="font-family:courier new,courier,monospace;"><a href="http://drupalcontrib.org/api/drupal/contributions!rules!rules.api.php/function/hook_rules_condition_info/7" target="_blank">hook_rules_condition_info()</a></span>&nbsp;in <span style="font-family:courier new,courier,monospace;">mymodule.rules.inc</span>&nbsp;:</p><p>&nbsp;</p>
<pre class="brush: php; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="mymodule.rules.inc">&lt;?php
/**
* Implements hook_rules_condition_info().
* Checks if the given user is admin.
*/
function mymodule_rules_condition_info() {
&nbsp; return array(
&nbsp;&nbsp;&nbsp; 'mymodule_user_is_admin' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'group' =&gt; t('My Module'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'label' =&gt; t('User is Admin'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'parameter' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'user' =&gt; array(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'type' =&gt; 'user',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'label' =&gt; t('User to test'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; )
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ),
&nbsp;&nbsp;&nbsp; ),
&nbsp; );
}
?&gt;
</pre>
<p>&nbsp;</p><p>Once again we see the familiar structure to declare what is about to happen. And just like in an Action, Rules is going to fire a function named after the array key, <span style="font-family:courier new,courier,monospace;">mymodule_user_is_admin</span>, with the parameters provided. Here we'll have a very simple function to check this, and it has to return <span style="font-family:courier new,courier,monospace;">TRUE</span> or <span style="font-family:courier new,courier,monospace;">FALSE</span>.&nbsp;</p><p>&nbsp;</p>
<pre class="brush: php; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="mymodule.module">&lt;?php
function mymodule_user_is_admin($user) {
  if ($user-&gt;uid == '1') {
    return TRUE;
  }
  return FALSE;
}
?&gt;
</pre>
<p>Great, so now your module is fully integrated with Rules, right? Well, you have to actually include the Rules logic with the module. Rules provides hook_rules_default_configuration for you to do this, but writing rules by hand is a pain. The shortcut solution is to build your Rules in the Rules GUI and export them, and paste the export into your code. <a href="http://drupalcontrib.org/api/drupal/contributions%21entity%21entity.module/7" target="_blank"><span style="font-family:courier new,courier,monospace;">entity_import()</span></a> is a handy function from the <a href="http://drupal.org/project/entity" target="_blank">EntityAPI</a> module you can use to translate the export format into the format Rules needs. Here's a sample <span style="font-family:courier new,courier,monospace;">(hook_rules_default_configuration</span> must be implemented in <span style="font-family:courier new,courier,monospace;">mymodule.rules_defaults.inc</span>):</p><p>&nbsp;</p>
<pre class="brush: php; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="mymodule.rules_defaults.inc">&lt;?php
/**
* Implements hook_default_rules_configuration().
*/
function mymodule_default_rules_configuration() {
&nbsp; $rules = array();
  $rules['rules_mymodule_behavior'] = entity_import(
    'rules_config',&nbsp;
    '............');
  return $rules;
}</pre>
<p>Note that the new rule name has to be prefixed with <span style="font-family:courier new,courier,monospace;">rules_</span>&nbsp;. &nbsp;That's all you need to do.</p><p>What's wonderful about these functions is how simple they are. Depending on how your module is written, adding Rules support could be as simple as adding calls to <span style="font-family: 'courier new', courier, monospace;">hook_rules_event_info()</span>, <span style="font-family: 'courier new', courier, monospace;">hook_rules_action_info()</span>, and <span style="font-family: 'courier new', courier, monospace;">hook_rules_condition_info()</span> that connect to existing functions already in your module. In 10 minutes or less, you can open your module up to much more flexibility than was ever possible before. And if you dare to prune the functionality out of your existing code, you can build it in the Rules GUI, export it, and have all your logic exposed and totally customizable in a further 15 minutes. That's less than half an hour, and you can start training your users to deal with their own problems. :)</p><p>Questions? Problems? Leave us a message in the comments...</p>
